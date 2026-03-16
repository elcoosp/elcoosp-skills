# AUD Escalation Runbook

Step-by-step procedures for every P0 and P1 scenario AUD can encounter.
AUD executes these procedures in order. Do not skip steps.

---

## RUNBOOK-01: Hardcoded Secret Detected in EL Artifact (P0)

**Trigger:** Pattern match on EL artifact finds API key, token, password, or
connection string.

```
Step 1 (T+0s):   Quarantine the EL artifact immediately
Step 2 (T+5s):   Emit circuit_breaker_triggered event, scope: this goal
Step 3 (T+10s):  Halt all downstream tasks in this goal (QA, TW, RM)
Step 4 (T+30s):  Send P0 PagerDuty alert to on-call human operator
                 Include: artifact_id, exact line/pattern found (redacted),
                          goal_id, task_id, suggested immediate action
Step 5 (T+60s):  Log full event to audit trail with all evidence
Step 6 (ongoing): Wait for human operator response
                  DO NOT: auto-remediate, suggest the secret be moved inline,
                  or allow EL to resubmit until human has confirmed the
                  secret is rotated (not just removed from code)

Human resolution path:
  - Human rotates the compromised credential
  - Human confirms rotation is complete
  - Human clears quarantine
  - EL resubmits artifact without secret
  - AUD validates clean artifact
  - AUD lifts circuit-breaker for this goal
```

**Why we wait for credential rotation before clearing quarantine:** Removing
a secret from code doesn't revoke it. The credential was in a draft artifact
that may have been logged, cached, or seen by other systems. Until it's rotated,
the exposure risk persists.

---

## RUNBOOK-02: Audit Log Integrity Failure (P0)

**Trigger:** Weekly hash chain re-computation finds a break in the chain.

```
Step 1 (T+0s):   Halt ALL pipelines immediately (scope: system-wide)
Step 2 (T+10s):  Send P0 PagerDuty alert to ALL designated security contacts
                 (not just on-call — this is a security incident, not an ops issue)
Step 3 (T+30s):  Record current state: which log entry is the first with an
                 invalid hash, and what sequence number
Step 4 (T+60s):  DO NOT attempt to repair the log — the break is evidence
                 Preserve the log in current state for forensic review
Step 5 (ongoing): Wait for human security team response
                  System remains halted until security team determines:
                  (a) cause of the break
                  (b) whether it represents tampering or a storage error
                  (c) whether any decisions made after the break point are valid

Note: AUD itself may have a bug that caused the break. This does not change
the procedure. Treat every audit log integrity failure as a potential security
incident until proven otherwise.
```

---

## RUNBOOK-03: CPO or RM Veto Override Detected Without Proper Log (P0)

**Trigger:** A deployment proceeds despite CPO-003 showing "blocked" outcome,
OR RM-001 missing a human override log entry when a hold was cleared.

```
Step 1 (T+0s):   Halt the deployment immediately if still in progress
Step 2 (T+10s):  Attempt to reverse any completed deployment steps (notify EL)
Step 3 (T+20s):  Send P0 PagerDuty alert: "Compliance control bypassed"
Step 4 (T+30s):  Log full chain of events: who cleared what, when, and how
Step 5 (T+60s):  Freeze the goal — no further tasks can proceed
Step 6 (ongoing): Require: identified human operator, their justification,
                  and explicit acknowledgement of risks accepted before
                  the goal can be unfrozen

Note: This is a controls failure, not just a process error. It must be treated
as such in the audit log and in any compliance reporting.
```

---

## RUNBOOK-04: Cost Runaway — Goal Exceeds 2× Budget (P1)

**Trigger:** SCH reports goal cost exceeds 2× CDR-001 estimated cost.

```
Step 1 (T+0s):   Pause all non-critical tasks in the affected goal
                 (critical = tasks on the critical path; non-critical = parallel tracks)
Step 2 (T+60s):  Produce cost analysis:
                 - Which tasks ran over estimate?
                 - By how much?
                 - Most likely cause (wrong capability tier? long context? loop?)
Step 3 (T+2min): Send P1 Slack alert to product owner with:
                 - Current spend vs estimate
                 - Per-task breakdown of overruns
                 - AUD's hypothesis for cause
                 - Three options: (a) continue, (b) descope, (c) pause for review
Step 4 (T+ongoing): Wait for human decision
                    CDR should be notified in parallel — CDR may have a re-plan

Critical: Do NOT automatically terminate running tasks. "Pause" means:
do not dispatch new tasks; let currently running tasks complete normally.
```

---

## RUNBOOK-05: Circuit-Breaker — Guardrail Violation Storm (P1)

**Trigger:** > 3 guardrail violations from the same agent within 5 minutes.

```
Step 1 (T+0s):   Pause all tasks assigned to the violating agent in this goal
Step 2 (T+30s):  Compile violation report:
                 - Which guardrail, how many times, what triggered each
                 - Is this the same violation repeated, or different violations?
                 - Is the pattern consistent with a specific input artifact?
Step 3 (T+2min): Send P1 Slack alert to product owner and CDR with violation report
Step 4 (T+30min): Auto-resume eligibility check:
                  - Have 30 minutes elapsed?
                  - Zero additional violations in those 30 minutes?
                  - Has human been notified (even without response)?
                  IF all three: resume
                  IF any no: extend pause; escalate to CDR for re-plan
```

---

## RUNBOOK-06: Compliance Gate Failure Before Deployment (P1)

**Trigger:** Pre-deployment compliance check finds missing or failing required artifact.

```
Step 1 (T+0s):   Emit compliance_gate_failed event
Step 2 (T+10s):  Halt RM task dispatch
Step 3 (T+30s):  Produce specific failure report:
                 - Which check failed (from the compliance gate checklist)
                 - What artifact is missing or in wrong state
                 - What needs to happen to pass the gate
Step 4 (T+2min): Notify RM and CDR with failure report
Step 5 (T+2min): Send P1 Slack alert to product owner if the release was
                 scheduled for imminent deployment (within 4 hours)
Step 6 (ongoing): Gate remains closed until the specific failure is resolved
                  and AUD re-runs the full compliance check (not just the
                  failed item — re-run the entire gate)
```

---

## RUNBOOK-07: Fabricated Artifact Reference Detected (P1)

**Trigger:** Content quality check finds an artifact_id referenced in an
artifact's content that does not exist in the Artifact Store.

```
Step 1 (T+0s):   Quarantine the artifact
Step 2 (T+30s):  Log hallucination indicator with full context:
                 - What artifact_id was referenced
                 - Where in the document it appeared
                 - What claim it was supporting
Step 3 (T+2min): Notify the producing agent and CDR:
                 "Artifact [ID] references [non-existent artifact_id] in support
                  of claim: [quote the claim]. This artifact is quarantined.
                  The producing agent must either (a) replace the reference with
                  a real artifact_id, or (b) remove the unsupported claim."
Step 4 (T+2min): Send P1 Slack alert if this is the second fabricated reference
                 from the same agent in one week (systematic pattern)
Step 5 (ongoing): Quarantine cleared only after agent resubmits clean version
                  AUD must re-validate the full artifact, not just the fixed section
```

---

## Human Response Expectations

When AUD fires an escalation, it expects a response within:

| Severity | Response SLA | If No Response by SLA |
|----------|-------------|----------------------|
| P0 | 15 minutes | Re-escalate via phone/secondary channel; system remains halted |
| P1 | 1 hour | Send reminder; after 2 hours, escalate to P0 if unresolved |
| P2 | 4 hours | Re-notify; log non-response |
| P3 | 24 hours | Log non-response; include in weekly report |

AUD tracks response times and includes them in the weekly report.
Consistent non-response to P1+ events is itself a governance finding.
