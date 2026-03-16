# Deployment Runbook — Step-by-Step Procedures by Stack

RM executes this runbook during every production deployment. Steps are ordered
and must be followed in sequence. Do not skip steps, even under time pressure.

---

## Pre-Deployment (T-30 minutes)

These steps happen 30 minutes before the deployment window opens.

```
[ ] 1. Confirm RM-001 checklist is 100% complete. If any item is false, STOP.
        Do not proceed to deployment until resolved or human override obtained.

[ ] 2. Notify all stakeholders:
        "Deployment beginning at [HH:MM UTC]. ETA: [duration].
        On-call: [name and contact]. Status updates in #deployments."

[ ] 3. Confirm on-call contact is available and knows they are on-call.
        Do not deploy if on-call contact is unreachable.

[ ] 4. Check current production error rate and P95 latency.
        Record as baseline: errors/min = N, P95 = Nms.
        If error rate is elevated above normal: STOP. Investigate before deploying.

[ ] 5. Verify the deployment window is still low-traffic.
        Check traffic dashboard. If traffic is unexpectedly high: delay deployment.

[ ] 6. Confirm rollback procedure is ready to execute.
        The rollback steps should be open in a separate terminal/tab — not looked up
        at the moment they're needed.
```

---

## Deployment: Docker/Kubernetes Stack

```
[ ] 7. Tag the release in Git:
        git tag -a v{VERSION} -m "Release v{VERSION} — RM-001-{release-id}"
        git push origin v{VERSION}

[ ] 8. Trigger the deployment pipeline (CI/CD deploy job, not manual):
        Confirm the pipeline is running against the tagged commit, not HEAD.

[ ] 9. Watch the pipeline execute. Do not leave the deployment unattended.
        At each stage (build → test → staging deploy → smoke test → prod deploy),
        verify the stage passes before the next begins.

[ ] 10. After staging smoke tests pass:
         Manually verify the critical path in staging:
         - Can a user log in?
         - Does the feature being deployed work?
         - Is the health endpoint returning 200?

[ ] 11. Approve the production promotion gate (if using manual approval in pipeline).

[ ] 12. Watch the rolling update progress:
         kubectl rollout status deployment/product-api -n production
         Expected: "deployment product-api successfully rolled out"
         If it stalls for > 5 minutes: see Rollback Triggers below.

[ ] 13. After rollout completes: run production smoke tests.
         See Smoke Test Checklist below.
```

---

## Deployment: Database Migrations

If this release includes database migrations, follow this sequence:

```
[ ] A. Take a manual database snapshot before running migrations.
        Record the snapshot ID in RM-001.

[ ] B. Run migrations in a maintenance window or using backward-compatible
        migration strategy (expand-contract pattern):
        Phase 1 — Expand: Add new column (nullable) — deploy with this column ignored by code
        Phase 2 — Migrate: Backfill data
        Phase 3 — Contract: Make column required; remove old column — only after all pods updated

[ ] C. Verify migration succeeded:
        SELECT COUNT(*) FROM schema_migrations WHERE version = '{migration_version}';
        Expected: count = 1

[ ] D. Verify the migration is reversible (rollback test was in RM-001 — confirm it was actually run).
```

---

## Smoke Test Checklist (Post-Deployment)

Run immediately after the deployment completes. Failures trigger rollback.

```
[ ] Health check: GET /health → 200 OK
[ ] Authentication: POST /auth/login with test credentials → valid JWT returned
[ ] Core feature smoke test 1: [specific to this release — defined in RM-001]
[ ] Core feature smoke test 2: [specific to this release]
[ ] Database connectivity: GET /health/db → connected
[ ] Cache connectivity: GET /health/cache → connected
[ ] Payment processing: [if applicable] Stripe test payment → success
[ ] Background jobs: confirm job queue is processing (not backed up)
[ ] Error rate: confirm errors/min ≤ baseline from step 4
[ ] P95 latency: confirm P95 ≤ baseline × 1.2 (allow 20% degradation before investigating)
```

---

## Rollback Triggers

Rollback immediately (do not wait, do not investigate) if any of the following occur:

```
🔴 Error rate > 5× baseline
🔴 P95 latency > 3× baseline
🔴 Any smoke test failure
🔴 Health check returning non-200
🔴 Payment processing failures
🔴 Authentication failures
🔴 Database connection errors
🔴 Deployment stalled > 10 minutes with no progress
```

Triggers that warrant investigation before rollback (judgment call):

```
🟡 Error rate 2–5× baseline (elevated but not critical)
🟡 P95 latency 1.5–3× baseline (degraded but not broken)
🟡 Spike in specific error type not seen before (may be data issue, not code)
```

---

## Rollback Procedure: Docker/Kubernetes

```
[ ] 1. Announce rollback:
        "ROLLBACK IN PROGRESS. Reason: [specific trigger]. ETA: 5 minutes."

[ ] 2. Roll back to previous deployment:
        kubectl rollout undo deployment/product-api -n production

[ ] 3. Watch rollback progress:
        kubectl rollout status deployment/product-api -n production

[ ] 4. Confirm rollback complete:
        kubectl rollout history deployment/product-api -n production
        Verify the REVISION column shows the previous version is active.

[ ] 5. Run smoke tests again on the rolled-back version.
        All must pass before declaring the rollback complete.

[ ] 6. Announce rollback complete:
        "ROLLBACK COMPLETE. Production is stable on v{PREVIOUS_VERSION}.
        Post-mortem scheduled for [time]."

[ ] 7. Log the rollback in RM-001 with: timestamp, trigger, duration.

[ ] 8. Notify CDR and human operator.
```

---

## Rollback Procedure: Database Migration

If a migration caused the issue:

```
[ ] 1. Run the down migration:
        [Migration tool command] down 1  (e.g., flyway undo, migrate down 1)

[ ] 2. Verify the migration was reversed:
        SELECT version FROM schema_migrations ORDER BY version DESC LIMIT 5;
        The rolled-back version should be absent.

[ ] 3. Restore from snapshot if down migration is not available or fails:
        This is destructive — any data written after the migration is lost.
        Confirm with human operator before proceeding.
        aws rds restore-db-instance-from-db-snapshot \
          --db-instance-identifier product-prod-restored \
          --db-snapshot-identifier {snapshot-id-from-step-A}
```

---

## Post-Deployment Monitoring (T+2 hours minimum)

```
Monitor the following for at least 2 hours after a successful deployment:
  - Error rate: should remain at or near baseline
  - P95 latency: should remain at or near baseline
  - Background job queue depth: should not grow unboundedly
  - Database connection pool usage: should not approach maximum
  - Memory usage: watch for slow leaks in new code

For major releases, monitor for 4 hours.
If any metric trends upward consistently: investigate before closing the deployment window.
```

---

## Closing the Deployment

After the monitoring window:

```
[ ] Announce deployment complete:
    "Deployment v{VERSION} complete. Monitoring clean for [N] hours.
    Deployment window closed."

[ ] Produce RM-002 (Release Notes) within 24 hours.

[ ] Close the rollback window: once stable, document in RM-001 that the
    rollback window is closed. (After this point, fixing forward is the
    standard approach rather than rollback.)
```
