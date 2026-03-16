# DAG Examples for Common Goal Types

Reference these patterns when decomposing goals. Adapt to the specific goal —
don't copy blindly. The value is in understanding the dependency structure.

---

## Pattern 1: New Feature (Standard)

**Goal type:** "Build and ship feature X"
**Typical node count:** 6–8
**Critical path:** PM → SA → CPO → EL → QA → RM

```
T1: PM — Write PRD (PM-003)              deps: []
T2: SA — Architecture Blueprint (SA-001)  deps: [T1]
T3: CPO — Security assessment (CPO-003)   deps: [T2]       ← PARALLEL with T4
T4: DX — Feature design spec (DX-002)     deps: [T1]       ← PARALLEL with T3
T5: EL — Implementation (EL-001)          deps: [T2, T3]
T6: QA — Test plan + execution (QA-001)   deps: [T5]
T7: TW — Documentation (TW-003)           deps: [T5]       ← PARALLEL with T6
T8: RM — Release checklist (RM-001)       deps: [T6, T7]
```

**Notes:**
- T3 and T4 run in parallel — CPO reviews the architecture while DX designs the UI
- T7 (TW) runs in parallel with T6 (QA) — docs don't block testing
- If TW is not yet activated, T7 is omitted and T8 depends only on T6

---

## Pattern 2: New Integration

**Goal type:** "Integrate with external system X"
**Typical node count:** 7–9
**Critical path:** PM → SA → CPO → EL → QA → RM
**Key difference:** CPO DPIA is required before any design begins if PII is involved

```
T1: PM — Integration requirements PRD (PM-003)       deps: []
T2: CPO — DPIA if PII involved (CPO-001)             deps: [T1]   ← GATE
T3: SA — Integration architecture + ADR (SA-001)     deps: [T1, T2]
T4: CPO — Architecture security assessment (CPO-003)  deps: [T3]   ← GATE
T5: EL — Integration implementation (EL-001)          deps: [T3, T4]
T6: EL — Integration Hub connector config             deps: [T3]   ← PARALLEL T4
T7: QA — Integration test suite (QA-001)              deps: [T5, T6]
T8: TW — Integration docs (TW-003)                    deps: [T5]   ← PARALLEL T7
T9: RM — Release (RM-001)                             deps: [T7, T8]
```

**Notes:**
- T2 (DPIA) is a hard gate. If CPO determines high residual risk, human review
  required before T3 can begin. CDR must wait; do not route around the gate.
- T6 (connector config) can start once T3 is done — no need to wait for CPO T4
  since connector config itself doesn't implement business logic

---

## Pattern 3: Go-To-Market Launch

**Goal type:** "Launch product/feature to market"
**Typical node count:** 8–10
**Critical path:** PM → MK → SAE → RM
**Key difference:** Two parallel tracks (product readiness + GTM readiness) must
both complete before RM can release

```
Track A — Product readiness:
T1: PM — Update product context doc (PM-001)    deps: []
T2: EL — Verify all features shipped            deps: [T1]
T3: QA — Regression test suite (QA-001)         deps: [T2]
T4: TW — Public docs complete (TW-002)           deps: [T2]

Track B — GTM readiness:
T5: MK — GTM plan (MK-001)                       deps: [T1]
T6: MK — Campaign assets ready                   deps: [T5]
T7: SAE — Sales playbook updated (SAE sales ref)  deps: [T5]

Gate — Both tracks must complete:
T8: RM — Release checklist (RM-001)               deps: [T3, T4, T6, T7]
```

**Notes:**
- T1–T4 and T5–T7 run fully in parallel
- A common CDR mistake is to sequence GTM after product — they should be parallel
- RM is the synchronisation point. Do not create a pre-RM "approval" node — RM IS the approval

---

## Pattern 4: Customer Health Intervention

**Goal type:** "Recover at-risk enterprise account"
**Typical node count:** 4–6
**Critical path:** CSH → PM → CSH
**Key difference:** Begins with CSH assessment, feeds back into PM for product action

```
T1: CSH — Account health assessment (CSH-004)   deps: []
T2: PM — Product gap analysis from CSH-004       deps: [T1]   ← CDR judgment call
T3: CSH — Executive sponsor outreach plan        deps: [T1]
T4: SAE — Commercial options assessment          deps: [T1]
T5: CSH — Recovery plan (CSH-004 updated)        deps: [T2, T3, T4]
```

**Notes:**
- T2, T3, T4 all run in parallel after T1
- PM's involvement (T2) is a CDR judgment call — only include if the churn signal
  is clearly product-driven. If it's relationship or pricing, PM is not needed.

---

## Pattern 5: Compliance Remediation

**Goal type:** "Fix security finding / compliance gap"
**Typical node count:** 5–7
**Critical path:** CPO → SA → EL → CPO → RM
**Key difference:** CPO owns the bookends; urgency may compress timelines

```
T1: CPO — Finding analysis + remediation plan    deps: []
T2: SA — Architecture fix design (ADR required)  deps: [T1]
T3: EL — Implement remediation (EL-001)          deps: [T2]
T4: QA — Security-focused test plan (QA-001)     deps: [T3]
T5: CPO — Remediation verification (CPO-002 upd) deps: [T4]
T6: RM — Release (RM-001)                        deps: [T5]
```

**Notes:**
- For P0 security findings, CDR should set priority: critical and flag for
  immediate human operator awareness before even completing decomposition
- Do not skip T5 (CPO verification) to save time. A remediation that CPO hasn't
  verified is not a remediation.

---

## Common CDR Mistakes to Avoid

| Mistake | Consequence | Correct Approach |
|---------|------------|-----------------|
| Making RM a dependency mid-DAG | RM blocks flow; RM is only an end-gate | RM only appears as the final release node |
| CPO review after EL implementation | Rework cost; may require reverting code | CPO architecture review always precedes EL |
| Single sequential chain (no parallelism) | Unnecessary time loss | Actively look for tasks whose inputs are independent |
| Over-splitting EL work into 5+ micro-tasks | Coordination overhead exceeds value | EL tasks should be feature-level, not function-level |
| Forgetting AUD as implicit listener | AUD always monitors; don't make it a node | AUD is not a task node — it's infrastructure |
| Including SCH as a node | SCH manages execution, not domain work | SCH is never a task node in a domain DAG |
