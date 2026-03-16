# Rollback Playbooks by Failure Type

RM uses these playbooks when a deployment must be reversed. Each playbook
covers a specific failure scenario with targeted recovery steps.

The goal of a rollback is to restore service, not to diagnose the problem.
Diagnose after service is restored.

---

## Playbook 1: Application Code Rollback (No DB Migration)

**When to use:** New code is causing errors or performance degradation.
No database schema changes were included in this release.

**Recovery time objective:** < 5 minutes from decision to stable.

```
1. kubectl rollout undo deployment/{service-name} -n production
2. Watch: kubectl rollout status deployment/{service-name} -n production
3. Confirm: GET /health returns 200
4. Run smoke tests
5. Announce: "Rollback complete. Stable on v{N-1}."
6. Log in RM-001 with timestamp and trigger
```

No database action required. This is the simplest rollback.

---

## Playbook 2: Application Code Rollback (With DB Migration — Backward Compatible)

**When to use:** Release included a migration designed using expand-contract
pattern (new column added as nullable, no columns removed). The application
code is failing but the schema change is safe.

**Strategy:** Roll back the application code. Leave the schema change in place
(it's backward compatible). The v{N-1} application code works with the new schema.

```
1. kubectl rollout undo deployment/{service-name} -n production
2. Watch and confirm as Playbook 1
3. DO NOT run a database down migration — the schema change is backward compatible
4. Verify v{N-1} code works correctly with the new schema by checking:
   - Are reads working? (The new nullable column returns null for old rows — expected)
   - Are writes working? (New rows don't set the new column — expected)
5. Document: "Application rolled back to v{N-1}. Schema change ADR-NNN retained.
   Will complete migration in next release cycle."
```

---

## Playbook 3: Application + Database Rollback (Breaking Schema Change)

**When to use:** Release included a breaking schema change (column removed,
column renamed, NOT NULL added without default, data type changed) AND the
application is failing.

**Warning:** This playbook may involve data loss if any writes occurred after
the migration. Confirm with human operator before executing.

**Recovery time objective:** 15–30 minutes (longer due to DB operations).

```
1. STOP new writes immediately:
   a. Put the application in maintenance mode (return 503 to all requests)
   OR
   b. Scale down the deployment: kubectl scale deployment/{service} --replicas=0

2. Assess data loss window:
   - When did the migration run?
   - How many writes occurred between migration and decision to rollback?
   - Are those writes recoverable from application logs or event sourcing?
   - Get human operator confirmation on acceptable data loss.

3a. If down migration is available and tested:
    Run: [migration tool] down 1
    Verify: SELECT MAX(version) FROM schema_migrations; → previous version
    Restore application: kubectl rollout undo deployment/{service-name}
    Resume traffic

3b. If down migration is NOT available (or migration tool doesn't support undo):
    Restore from snapshot taken before migration (Playbook 1 in deployment-runbook.md)
    Note: ALL data written after snapshot is lost
    Confirm with human operator
    aws rds restore-db-instance-from-db-snapshot ...
    Update application DB_URL to point to restored instance
    Roll back application code
    Resume traffic

4. Verify: smoke tests pass; data integrity spot-check
5. Announce: "Rollback complete. Data loss window: [timestamp A] to [timestamp B].
   [N] records affected. Human operator informed."
6. Trigger post-mortem immediately
```

---

## Playbook 4: Partial Deployment Failure (Canary / Rolling Update Stuck)

**When to use:** A rolling update has partially completed. Some pods are
running the new version, some are running the old version, and the new pods
are failing their health checks.

```
1. Check pod status:
   kubectl get pods -n production -l app={service-name}
   Look for pods in CrashLoopBackOff or 0/1 Ready state

2. Check new pod logs:
   kubectl logs -n production {failing-pod-name} --previous

3. If the issue is clear and fixable in < 5 minutes: fix it (e.g., wrong
   environment variable, misconfigured secret mount)

4. If not fixable quickly: rollback:
   kubectl rollout undo deployment/{service-name} -n production

5. The rollback will terminate the new pods and restore all pods to v{N-1}

6. Traffic continues to flow through the old pods during the rollback —
   users may experience errors from the bad pods until they are terminated
   (typically 30–60 seconds)
```

---

## Playbook 5: Third-Party Integration Rollback

**When to use:** The new release changed integration behaviour (new API version,
changed webhook handling, modified auth flow) and the integration is now broken.

```
1. Assess scope:
   - Which integration is affected?
   - Is it inbound (we receive data) or outbound (we send data)?
   - What data may have been lost or duplicated during the failure window?

2. Disable the integration immediately:
   - Open the circuit breaker manually in the Agent State Store
   SET sch:model-health:integration-name '{"fallback_active": true}'
   OR
   - Set integration to degraded mode in the application config

3. Rollback the application code (Playbook 1 or 2)

4. Re-enable the integration after rollback confirmed stable

5. Replay any failed events from the DLQ in order:
   - Review DLQ entries for the affected integration
   - Replay in chronological order
   - Verify each replay processed correctly before replaying the next

6. Reconcile data:
   - Compare integration system's records with our records for the failure window
   - Identify any discrepancies
   - Resolve manually or via a targeted reconciliation script
   - Document discrepancies in the post-mortem
```

---

## Rollback Decision Matrix

Use this when unsure whether to rollback or fix forward:

| Condition | Decision |
|-----------|---------|
| Error rate > 5× baseline | Rollback immediately |
| Any P0 bug in new code affecting core features | Rollback immediately |
| Smoke test failure | Rollback immediately |
| Database migration caused data corruption | Rollback + snapshot restore |
| Error rate 2–5× baseline | Investigate 5 min; rollback if no quick fix |
| Single non-critical endpoint failing | Fix forward if < 30 min fix |
| Performance degradation < 2× baseline | Monitor; fix forward |
| Minor UI bug | Fix forward in next deploy |

**Default to rollback** when uncertain. A rollback is always reversible.
A forward deploy with a live bug may not be.

---

## Post-Rollback Checklist

After any rollback, before the incident is considered closed:

```
[ ] Smoke tests passing on rolled-back version
[ ] Error rate back to baseline
[ ] P95 latency back to baseline
[ ] Human operator notified
[ ] CDR notified (to update goal status)
[ ] AUD log entry created with full rollback timeline
[ ] Post-mortem scheduled within 48 hours
[ ] RM-001 updated with rollback record
```

A rollback is not a failure. It is the correct response to a failed deployment.
The failure is deploying code that wasn't ready — and that's a process improvement
discussion, not a blame discussion.
