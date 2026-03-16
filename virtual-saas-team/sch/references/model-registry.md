# Model Registry: How SCH Reads and Uses It

The model registry is the single source of truth for which model runs at which
capability tier. SCH reads it at runtime. It is never hardcoded into agent specs.

---

## Registry Location

```
/config/model-registry.yaml
```

Stored in the Artifact Store as a configuration artifact. Not versioned like
domain artifacts — it is updated in place when models change. Controlled by SA.

---

## Current Registry (as of 2026-03)

```yaml
registry_version: "2026-03"
last_updated: "2026-03-16"
updated_by: "SA"

tiers:
  reasoning_heavy:
    primary: claude-opus-4-6
    fallback: claude-sonnet-4-6
    max_tokens: 8192
    timeout_seconds: 120
    cost_per_1k_tokens_usd: 0.015
    rate_limit_tokens_per_minute: 40000

  reasoning_balanced:
    primary: claude-sonnet-4-6
    fallback: claude-haiku-4-5-20251001
    max_tokens: 4096
    timeout_seconds: 60
    cost_per_1k_tokens_usd: 0.003
    rate_limit_tokens_per_minute: 80000

  fast:
    primary: claude-haiku-4-5-20251001
    fallback: claude-haiku-4-5-20251001
    max_tokens: 2048
    timeout_seconds: 15
    cost_per_1k_tokens_usd: 0.00025
    rate_limit_tokens_per_minute: 200000

  deterministic:
    primary: rule-engine
    fallback: rule-engine
    max_tokens: null
    timeout_seconds: 5
    cost_per_1k_tokens_usd: 0
    rate_limit_tokens_per_minute: null

review_cadence: quarterly
next_review: "2026-06-30"
```

---

## How SCH Reads and Caches the Registry

```
Cache TTL: 5 minutes
On cache miss or expiry:
  1. Read /config/model-registry.yaml from Artifact Store
  2. Validate: registry_version field is present
  3. Validate: all four tiers (reasoning_heavy, reasoning_balanced, fast, deterministic) are present
  4. If validation fails: use last valid cached copy + alert AUD (P1)
  5. If no valid cache exists: HALT all dispatching + alert AUD (P0)
```

Never hold the registry in process memory beyond the 5-minute TTL. Registry
updates (e.g., a model being deprecated mid-day) must propagate within 5 minutes.

---

## Model Health Tracking

SCH tracks model health per tier in the Agent State Store:

```
key: model-health:{tier}
value:
  primary_consecutive_failures: 0
  last_failure_timestamp: null
  fallback_active: false
```

**Fallback triggers:**
- 3 consecutive failures from the primary model within 10 minutes
- Primary model returns HTTP 529 (overloaded) twice in 5 minutes
- Primary model average latency exceeds timeout_seconds × 0.8

**Fallback reset:**
- Primary model succeeds 3 consecutive times
- SCH resets to primary and logs the recovery to AUD

---

## Updating the Registry

Only SA updates the registry. SA produces a change proposal; CDR approves it
for models that affect in-flight goals; SA writes the file.

**When a model is deprecated mid-execution:**
1. SA updates the registry (new primary or fallback)
2. SCH picks up the change within 5 minutes (cache TTL)
3. All subsequent task dispatches use the updated model
4. In-flight tasks (already dispatched) are not interrupted — they complete
   with the model they were dispatched with
5. SCH logs the model change event to AUD

---

## Cost Estimation Reference

For CDR-001 cost estimates, CDR uses these rates (from the registry):

| Tier | Est. tokens/hour | Cost/hour |
|------|-----------------|-----------|
| `reasoning_heavy` | ~50k | ~$0.75 |
| `reasoning_balanced` | ~30k | ~$0.09 |
| `fast` | ~20k | ~$0.005 |
| `deterministic` | 0 | $0.00 |

These are rough estimates for planning. Actual costs vary by task complexity.
SCH tracks actual tokens and reports variance to CDR at goal completion.
