# Integration Patterns Reference

SA uses these patterns when specifying external integrations in SA-001.
Every integration in an Architecture Blueprint must declare its pattern,
its auth method, and its failure behaviour. This file is the pattern library.

---

## Pattern Selection Guide

| If the integration needs... | Use pattern |
|-----------------------------|------------|
| Real-time bidirectional sync | Event stream |
| Push notifications from external system | Webhook |
| On-demand reads/writes | REST API |
| Complex queries across related data | GraphQL |
| Bulk data transfer (ETL, reports) | File transfer |

---

## Pattern: REST API

**Use for:** CRM reads/writes, payment API calls, analytics data pushes, any CRUD operation against an external system.

**SA-001 integration block:**
```yaml
- system_name: Salesforce
  integration_pattern: rest
  auth_method: oauth2_client_credentials
  data_direction: bidirectional
  base_url: https://{instance}.salesforce.com/services/data/v59.0
  rate_limits:
    requests_per_day: 100000
    concurrent_requests: 25
  failure_behavior:
    on_timeout: retry with exponential backoff (see retry policy)
    on_4xx: fail fast — log error, do not retry
    on_5xx: retry up to 3 times, then DLQ
    on_circuit_open: return cached last-known value if available; else surface degraded state to user
  circuit_breaker_config:
    failure_threshold: 5
    window_seconds: 60
    half_open_probe_seconds: 30
    re_close_successes: 3
```

**Retry policy (applies to all REST integrations):**
```
Retryable:     429, 500, 502, 503, 504, connection timeout
Non-retryable: 400, 401, 403, 404 — log and fail immediately
Max retries:   5
Backoff:       exponential with ±20% jitter
               attempt 1: 1s, attempt 2: 2s, attempt 3: 4s, attempt 4: 8s, attempt 5: 16s
Cap:           120 seconds
```

---

## Pattern: Webhook

**Use for:** Receiving events pushed by external systems — payment confirmations, CRM updates, CI/CD status callbacks.

**Critical design rule:** Webhooks must be idempotent. The same event can be delivered more than once. The receiver must check for duplicate `event_id` before processing.

**SA-001 integration block:**
```yaml
- system_name: Stripe
  integration_pattern: webhook
  auth_method: hmac_signature_verification
  data_direction: inbound
  endpoint: /webhooks/stripe
  events_subscribed:
    - payment_intent.succeeded
    - payment_intent.failed
    - subscription.updated
    - subscription.deleted
  idempotency:
    dedup_key: stripe_event_id
    dedup_window_hours: 24
    dedup_store: Redis
  failure_behavior:
    on_processing_error: return 500 to Stripe (triggers retry from their side)
    on_duplicate: return 200 immediately without reprocessing
    max_processing_time_ms: 5000  # Return 200 fast; process async
  security:
    verify_signature: true
    secret_location: /secrets/webhooks/stripe
    reject_unverified: true
```

**Webhook receipt pattern:**
```
1. Receive event → verify HMAC signature → return 200 immediately
2. Enqueue event payload to internal message queue
3. Background worker dequeues → checks idempotency key → processes
4. Mark event as processed in dedup store
```
Never process inline in the webhook handler. Always return 200 fast and process async.

---

## Pattern: Event Stream

**Use for:** High-volume, real-time, bidirectional data flows. Usage telemetry, activity feeds, audit event pipelines.

**SA-001 integration block:**
```yaml
- system_name: PostHog
  integration_pattern: event_stream
  auth_method: api_key
  data_direction: outbound
  transport: https_batch
  batch_config:
    max_events_per_batch: 1000
    max_batch_size_kb: 512
    flush_interval_seconds: 30
    flush_on_shutdown: true
  failure_behavior:
    on_flush_failure: buffer to local disk queue; retry on next interval
    on_disk_queue_full: drop oldest events; emit metric: events_dropped_total
    on_circuit_open: buffer locally; resume when circuit closes
  ordering_guarantee: best_effort  # not guaranteed across batches
```

---

## Pattern: GraphQL

**Use for:** When REST would require multiple round-trips to assemble a response, or when clients need to specify exact field sets (reduces over-fetching).

**SA-001 integration block:**
```yaml
- system_name: GitHub
  integration_pattern: graphql
  auth_method: github_app_jwt
  data_direction: bidirectional
  endpoint: https://api.github.com/graphql
  rate_limits:
    points_per_hour: 5000  # GitHub GraphQL rate limit
    max_query_complexity: 500
  failure_behavior:
    on_complexity_exceeded: decompose into multiple simpler queries
    on_rate_limit: backoff until reset time from X-RateLimit-Reset header
    on_partial_errors: treat as failure if any requested field is null due to error
  caching:
    strategy: time-based
    ttl_seconds: 300  # Cache repo metadata; never cache live PR status
```

---

## Pattern: File Transfer

**Use for:** Bulk data exports, scheduled ETL jobs, report delivery, data migrations.

**SA-001 integration block:**
```yaml
- system_name: accounting-erp
  integration_pattern: file_transfer
  auth_method: sftp_key
  data_direction: bidirectional
  transport: sftp
  schedule: daily at 02:00 UTC
  format: csv_with_header
  failure_behavior:
    on_transfer_failure: retry 3 times at 15-minute intervals; alert on third failure
    on_parse_failure: quarantine file to /data/failed-imports/; alert immediately
    on_partial_success: process valid rows; quarantine invalid rows with row-level error log
  idempotency:
    dedup_key: filename + sha256_checksum
    reprocess_policy: skip if already processed successfully
  monitoring:
    alert_if_no_file_by: 04:00 UTC  # Expected by 02:30; alert if 90 min late
```

---

## Circuit Breaker State Machine

All Integration Hub connectors implement this state machine. SA specifies the thresholds; the hub implements the logic.

```
                    failure_threshold reached
CLOSED ──────────────────────────────────────────► OPEN
  ▲                                                   │
  │ 3 consecutive                   half_open_probe_  │
  │ successes                       seconds elapsed   │
  │                                                   ▼
  └──────────────── HALF-OPEN ◄───────────────────────
                        │
                        │ probe request fails
                        ▼
                      OPEN (reset timer)
```

**State meanings:**
- **CLOSED** — normal operation; all requests pass through
- **OPEN** — all requests fail immediately without calling the external system; returns cached value or degraded-mode response
- **HALF-OPEN** — one probe request allowed; success → CLOSED; failure → OPEN

**Circuit breaker state is stored in the Agent State Store (Redis), not in-process.** This means the state survives process restarts and is shared across multiple instances of the same connector.

---

## Dead-Letter Queue (DLQ) Pattern

Every integration that uses async processing (webhooks, event streams, file transfer) needs a DLQ.

```
Normal flow:
  Event → Queue → Consumer → Processed ✓

Failure flow (after max retries):
  Event → Queue → Consumer → Fails → Retry 1
                                   → Retry 2
                                   → Retry 3
                                   → DLQ (with: original payload, all attempt timestamps, last error message)

DLQ review:
  Human reviews → Fix root cause → Manually replay from DLQ
  OR
  Human reviews → Discard as invalid
```

**DLQ alert thresholds** (configure per integration based on criticality):
- Payment webhooks: alert immediately on any DLQ entry
- CRM sync events: alert if DLQ depth > 10
- Analytics events: alert if DLQ depth > 1000

---

## Auth Method Quick Reference

| Method | When to use | Where secrets live |
|--------|------------|-------------------|
| `oauth2_client_credentials` | Server-to-server; no user involved | Vault at `/secrets/integrations/{system}` |
| `oauth2_authorization_code` | User-delegated access (user authorises the app) | Vault; refresh token stored encrypted |
| `api_key` | Simple integrations; single credential | Vault; never in config files |
| `hmac_signature_verification` | Webhook signature verification | Vault; verified on receipt |
| `github_app_jwt` | GitHub App installations | Private key in Vault; JWT generated at runtime |
| `sftp_key` | File transfer via SFTP | Private key in Vault |
| `mtls` | Restricted data class; highest security | Client cert in Vault |

**Rule:** Secrets are always in the vault. The path to the secret appears in SA-001. The secret value never appears in any artifact, config file, or log.
