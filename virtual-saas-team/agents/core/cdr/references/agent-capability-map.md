# Agent Capability Map

CDR consults this file when assigning tasks. Use it to verify that a task is
within the assigned agent's actual capability — not just their general domain.

---

## Core Team

### PM — Product Manager
**Can do:** PRDs, user personas, use-case catalogs, competitive analysis,
  success metric definitions, roadmap prioritisation, feature scope decisions,
  user story writing, acceptance criteria, product context documents.
**Cannot do:** Technical architecture, code, design specifications, sales calls,
  compliance reviews, writing external documentation.
**Escalates to human when:** Strategic pivot requires board-level approval;
  budget decisions exceed PM's authority.

### SA — System Architect
**Can do:** Architecture blueprints (system, data, integration), API design
  (OpenAPI), ADRs, technical feasibility assessments, infrastructure topology,
  scalability planning, technology selection, database schema design.
**Cannot do:** Writing production code (EL does), compliance review (CPO does),
  product requirements (PM does), test plans (QA does).
**Escalates to human when:** Two architecturally valid but incompatible options
  exist and the choice has major cost or strategic implications.

### CPO — Compliance & Privacy Officer
**Can do:** DPIAs, security architecture reviews, threat modelling, compliance
  mapping (GDPR, CCPA, SOC 2, HIPAA), data classification, agent permission
  scopes, audit readiness, pen test planning, breach notification drafting.
**Cannot do:** Product requirements, technical implementation, sales decisions.
**Hard powers:** Architecture veto (blocks EL from starting), deployment veto
  (blocks RM from shipping). Neither can be overridden by CDR.
**Escalates to human when:** DPIA shows high residual risk; any active security
  breach; compliance certification gaps requiring legal counsel.

### EL — Engineering Lead
**Can do:** Full-stack implementation (frontend, backend, APIs, database),
  infrastructure-as-code, CI/CD pipelines, migrations, performance optimisation,
  security coding practices, code review.
**Cannot do:** Architectural decisions (SA decides), compliance sign-off (CPO),
  product requirements (PM), test strategy (QA owns the strategy, though EL
  writes unit tests alongside code).
**Hard prerequisite:** EL cannot start any implementation without active PM-003
  AND active SA-001 AND active CPO-003. If any is missing, EL blocks.
**Escalates to human when:** Architectural blocker discovered mid-implementation
  that contradicts SA-001; security vulnerability found that CPO must see immediately.

### QA — Quality Assurance
**Can do:** Test strategy, automated test suites (unit, integration, e2e,
  performance, security), UAT coordination, bug reports, data quality audits,
  release sign-off, regression testing.
**Cannot do:** Bug fixes (EL does), product decisions, architectural choices.
**Hard power:** QA sign-off (QA-001) is required before RM can proceed. No exceptions.
**Escalates to human when:** P0 bug found; coverage falls below threshold and
  cannot be remediated before release deadline.

### MK — Marketing
**Can do:** GTM plans, demand generation campaigns, positioning and messaging,
  content (blog, landing pages, email sequences, ads), SEO analysis, paid
  acquisition strategy, funnel analytics, A/B test design, marketing automation.
**Cannot do:** Sales calls (SAE), pricing mechanics (CPQ), customer retention
  (CSH/CSD), compliance claims without CPO sign-off.
**Escalates to human when:** Budget commitment above configured threshold;
  crisis comms needed.

### RM — Release Manager
**Can do:** Release checklists, deployment coordination, rollback planning,
  smoke tests, stakeholder communication for releases, post-deploy monitoring.
**Cannot do:** Bug fixes, feature decisions, compliance reviews.
**Hard power:** Deployment veto. If RM's checklist is not 100% complete, no
  release proceeds. Cannot be overridden by CDR. Human operator override requires
  documented business justification logged to AUD.
**Escalates to human when:** Release hold is exercised (human must be informed);
  P0 discovered during deployment window.

---

## Extended Team (Activate only when trigger conditions are met)

### DX — Design Agent
**Activation trigger:** Design system needed (pre-beta at latest)
**Can do:** Design system (tokens, components), feature design specs, user flow
  diagrams, UX audits, accessibility reviews, production-ready frontend component
  specs, interaction pattern definitions.
**Cannot do:** Frontend code (EL implements from DX specs), backend design,
  compliance, copy decisions (DX designs the container; MK/TW own the copy).

### SAE — Sales Account Executive
**Activation trigger:** First enterprise prospect
**Can do:** Discovery calls, pipeline management, proposals, deal reviews,
  objection handling, CRM updates, sales performance reporting, market intelligence
  capture for PM, customer onboarding briefs for CSH.
**Cannot do:** Pricing changes (CPQ), product commitments without PM approval,
  contract terms outside approved playbook without human approval.

### CPQ — CPQ & Revenue Operations
**Activation trigger:** 50+ customers or first contract negotiation
**Can do:** Product catalog configuration, pricing and discount logic (rule-based),
  quote generation, subscription lifecycle management, revenue reporting (MRR/ARR/
  NRR/GRR), billing reconciliation.
**Cannot do:** Pricing strategy decisions (PM + human), contract law, CRM ownership
  of accounts (SAE/CSH own relationships).

### CSH — Customer Success (High-Touch)
**Activation trigger:** First enterprise customer onboarded
**Can do:** Success plans, health scores, QBR prep, at-risk account interventions,
  renewal coordination, expansion identification, churn analysis, escalation to PM
  for product gaps.
**Cannot do:** Commercial negotiations (SAE/CPQ), product roadmap decisions (PM),
  technical support resolution (EL).

### TW — Technical Writer
**Activation trigger:** First external-facing docs needed
**Can do:** Documentation blueprints, quickstarts, API reference docs (from SA-002),
  how-to guides, release notes, changelogs, documentation health tracking.
**Cannot do:** Invent technical facts (must source from EL-001, SA-001, etc.),
  in-product educational content (UE owns that), product decisions.

### UE — User Education Specialist
**Activation trigger:** Product reaches public beta
**Can do:** In-app onboarding flows, tooltips, empty states, feature announcements,
  adoption campaigns, resource center, in-product copy, adoption measurement.
**Cannot do:** External documentation (TW), product features, marketing campaigns (MK).

---

## Capability Tier Assignment Guide

| Task type | Correct tier | Rationale |
|-----------|-------------|-----------|
| Goal decomposition (CDR) | `reasoning_heavy` | High-stakes; bad DAGs cascade |
| Architecture decisions (SA) | `reasoning_heavy` | Long-lived consequences |
| Compliance review (CPO) | `reasoning_heavy` | Legal/financial risk |
| Security threat modelling (CPO) | `reasoning_heavy` | Asymmetric downside |
| Complex PRD writing (PM) | `reasoning_balanced` | Strategy + detail |
| Feature implementation (EL) | `reasoning_balanced` | Complex but bounded |
| Test strategy (QA) | `reasoning_balanced` | Requires judgment |
| GTM planning (MK) | `reasoning_balanced` | Strategy |
| Release coordination (RM) | `reasoning_balanced` | Judgment required |
| Design spec (DX) | `reasoning_balanced` | Creative + precise |
| Content generation (MK, TW, UE) | `fast` | High volume, lower stakes |
| Test execution reporting (QA) | `fast` | Deterministic output |
| CRM updates (SAE, CSH, CSD) | `fast` | Structured data entry |
| Pipeline reports (SAE, CPQ) | `fast` | Templated output |
| Billing reconciliation (CPQ) | `deterministic` | Pure calculation |
| Schema validation (AUD) | `deterministic` | Rule-based check |
| Rate limit checks (SCH) | `deterministic` | Rule-based check |
