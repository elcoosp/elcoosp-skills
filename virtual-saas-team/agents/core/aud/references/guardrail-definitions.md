# Guardrail Definitions — All Agents

AUD loads this file to know exactly what to check for each agent's outputs.
Each guardrail has: a machine-checkable condition, the evidence to look for,
and the action to take on violation.

---

## CDR — Commander Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| CDR-G01 | Every CDR-001 must pass all 5 DAG validation checks before dispatch | CDR-001 must contain field `validation_passed: true` | Quarantine CDR-001; halt SCH dispatch |
| CDR-G02 | No production deployment without active RM-001 | Any DAG node with name matching deploy/release/production must have RM-001 in its dependency list | Quarantine the DAG; P1 escalation |
| CDR-G03 | Every CDR decision logged to AUD | CDR emits an audit event for every goal received, DAG dispatched, and re-plan triggered | Missing event = P2 flag; pattern of missing events = P1 |
| CDR-G04 | Escalate to human after 2 failed resolution cycles | CDR-004 conflict log must not show > 2 CDR-only resolution attempts for same conflict | If 3rd attempt found: P1 escalation to human with conflict context |
| CDR-G05 | No single task > 80 estimated hours in CDR-001 | Scan all node.estimated_effort_hours in CDR-001 | Flag tasks > 80h as P2; block dispatch until CDR confirms or splits the task |
| CDR-G06 | Total DAG cost > $200 requires human approval | Sum estimated_cost_usd across all nodes | If > $200 and no human_approval field: quarantine CDR-001; P1 escalation |

---

## SCH — Scheduler Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| SCH-G01 | Never dispatch task with unmet dependencies | Before dispatch: verify all dependency artifact_ids are active in Artifact Store | If dispatched with unmet dep: halt that task; P1 log; notify CDR |
| SCH-G02 | Log execution metrics to AUD after every task | AUD expects an `execution_metric` event for every completed/failed task | Missing metric after task event: P2 flag |
| SCH-G03 | Escalate to CDR after 3 retries, never a 4th | task.retry_count must not exceed 3 in Agent State Store | If 4th dispatch attempt detected: halt; P1 escalation |
| SCH-G04 | Never exceed rate limits | tokens_used_current_minute must not exceed rate_limit_tokens_per_minute | If exceeded: halt dispatching for that tier; P1 log |
| SCH-G05 | Task queue state persisted to Agent State Store, not in-memory only | Spot-check: AUD reads Agent State Store directly and compares to SCH's reported state | If >10% discrepancy: P1 escalation; possible data integrity issue |

---

## PM — Product Manager Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| PM-G01 | Every feature requirement links to ≥1 metric and ≥1 user story | PM-003 must contain non-empty `success_metrics` and `user_stories` sections | Missing section: quarantine artifact |
| PM-G02 | No active PM-003 with unvalidated `must_validate` assumptions | Scan PM-003 `assumptions_to_validate` for any item with no `validation_method` or past `deadline` | Flag as P2; PM must update before EL scheduling |
| PM-G03 | No scope changes after SA has produced SA-001 without CDR approval | If PM-003 version N+1 has expanded scope vs version N after SA-001 is active: check for CDR approval event | Missing CDR approval: quarantine PM-003 v N+1; P1 flag |

---

## SA — System Architect Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| SA-G01 | CPO-003 must be active before any EL task is dispatched | In the DAG, any EL task must have CPO-003 artifact in its resolved dependencies | If EL task dispatched without CPO-003 active: halt EL task; P1 escalation |
| SA-G02 | Every SA-001 must have at least one ADR | SA-001 artifact must reference at least one ADR-NNN artifact in its `adrs` section | Missing ADR: quarantine SA-001 |
| SA-G03 | Every external integration must specify failure behavior | SA-001 `integration_architecture` section must have `failure_behavior` field populated for each integration | Missing field: quarantine SA-001 |
| SA-G04 | No single points of failure on critical path without documented mitigation | SA-001 must not contain any component on the critical path with no redundancy AND no mitigation plan | P2 flag; notify SA and CDR |

---

## CPO — Compliance & Privacy Officer Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| CPO-G01 | DPIA required for any PII-touching feature before scheduling | If PM-003 mentions personal data collection, sharing, or new processing: CPO-001 must be active | If missing: block feature from SCH scheduling; P1 escalation |
| CPO-G02 | High residual risk DPIA requires human legal review | CPO-001 risk_assessment must show: if any risk has likelihood × impact ≥ 15, `human_legal_review_required: true` | If high risk DPIA lacks human approval field: quarantine; P1 escalation |
| CPO-G03 | CPO security assessment (CPO-003) must be active before EL starts | See SA-G01 — CPO-G03 is the same gate enforced from CPO's side | Same action as SA-G01 |
| CPO-G04 | Compliance claims in MK artifacts require CPO sign-off | Scan MK artifacts for regulatory terms: SOC 2, GDPR, HIPAA, CCPA, compliant, certified | If found without CPO sign-off reference: quarantine; P1 escalation |

---

## EL — Engineering Lead Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| EL-G01 | No hardcoded secrets in any EL artifact | Pattern scan: look for patterns matching API keys, tokens, passwords, connection strings in EL-001 and any code artifacts | If found: quarantine immediately; P0 escalation |
| EL-G02 | EL cannot start without active PM-003 and SA-001 | EL-001 upstream_artifacts must reference both | If EL-001 is submitted without these references: quarantine; P1 flag |
| EL-G03 | All database migrations must have rollback procedure documented | EL-001 `database_changes.rollback_tested` must be true if migrations_list is non-empty | If false: quarantine EL-001; notify QA and RM |
| EL-G04 | Test coverage ≥ 80% for new code | EL-001 `testing_summary.unit_coverage_pct` must be ≥ 80 | If < 80: quarantine EL-001; P2 flag; EL must add tests |
| EL-G05 | `known_limitations` field must not be blank | EL-001 `known_limitations` section must be non-empty | If empty: quarantine with message "known_limitations cannot be blank — either document actual limitations or document why there are none" |

---

## QA — Quality Assurance Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| QA-G01 | QA-001 active before RM task is dispatched | RM task in CDR-001 must have QA-001 as a resolved dependency | If RM dispatched without QA-001: halt RM task; P1 escalation |
| QA-G02 | P0 bugs do not appear in completed tasks | Scan QA bug log for any QA-002 with priority P0 and status ≠ resolved | If found: halt all downstream tasks including RM; P0 escalation |
| QA-G03 | Data quality score ≥ 95% for data-touching releases | QA-001 must reference a data quality audit with score ≥ 95% if the release touches the database | If missing or below threshold: quarantine QA-001; P1 flag |

---

## MK — Marketing Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| MK-G01 | No marketing claims about unshipped features | Scan MK artifacts for feature claims; cross-reference against active EL-001 artifacts | If claim has no active EL-001 backing: quarantine; P1 flag |
| MK-G02 | Paid spend above threshold requires CDR approval | MK-001 channel_strategy budget fields: if any monthly_budget exceeds configured threshold, CDR approval event must be present | Missing approval: quarantine MK-001 spend plan; P1 flag |
| MK-G03 | A/B tests must have hypothesis, sample size, end date | Scan MK campaign artifacts for any experiment without all three fields | Missing field: P2 flag (warn, don't quarantine) |

---

## RM — Release Manager Agent

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| RM-G01 | RM-001 checklist 100% complete before deployment step | All boolean fields in RM-001 technical_checks and business_checks must be true | Any false field: halt deploy step; P1 escalation |
| RM-G02 | Human override of RM veto must be logged | If RM-001 shows a hold was cleared, a human_override field with operator_name and justification must be present | Missing override log: P0 — this is a controls failure |
| RM-G03 | Rollback procedure must be tested, not just written | RM-001 `rollback_procedure_tested` must be true with evidence of test | If false or missing evidence: quarantine RM-001; P1 flag |

---

## Cross-Agent (System-Level Guardrails)

| Guardrail ID | Condition | Evidence | Action on Violation |
|-------------|-----------|---------|---------------------|
| SYS-G01 | No direct agent-to-agent synchronous calls | Event Bus must be the only inter-agent communication channel | If direct call detected: P1 log; notify CDR and the two involved agents |
| SYS-G02 | No Restricted data accessed by non-approved agents | Cross-reference agent permissions.yaml against actual data access in artifacts | If violation: P0 escalation; halt affected pipeline |
| SYS-G03 | No routing around a quarantined artifact | No task should be dispatched that takes a quarantined artifact as input | If detected: halt task; P1 escalation; CDR guardrail violation logged |
| SYS-G04 | Audit log chain integrity maintained | Weekly hash chain re-computation must pass | If chain break: P0 escalation; halt all pipelines |
