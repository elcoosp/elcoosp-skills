# ADR Library — Examples and Anti-Patterns

SA writes an ADR for every significant technical decision. This file provides
annotated examples of well-written ADRs and common anti-patterns to avoid.

An ADR is not a meeting summary. It is a permanent record of a decision,
the problem it solved, and the trade-offs accepted. Future engineers (human
or agent) will read it when they want to understand why the system is the
way it is — often when something breaks.

---

## What Makes an ADR Good

- **The context section explains the pressure**, not just the situation. "We needed a database" is not context. "Read traffic was growing 40% MoM and our single PostgreSQL instance was hitting CPU limits at 10k concurrent users" is context.
- **The decision is a complete sentence.** "Use Redis" is not a decision. "Use Redis Cluster as our primary session store and ephemeral cache layer, replacing in-process memory caching" is a decision.
- **Alternatives considered are honest.** List options you actually evaluated, not strawmen. Include why each was rejected — especially if the rejected option looks obviously better to someone reading cold.
- **Consequences include negatives.** Every decision has a downside. An ADR with only positive consequences is either lying or incomplete.
- **The review date is real.** Set it 6–12 months out. When it arrives, CDR should re-evaluate whether the decision still holds.

---

## Example 1: Database Selection (Good ADR)

```markdown
---
adr_id: ADR-001
title: Use PostgreSQL for all transactional data with per-tenant schema isolation
date: 2026-04-01
status: accepted
supersedes: null
---

## Context

The product is a multi-tenant SaaS. We need a transactional database that:
- Supports row-level isolation between tenants
- Is ACID-compliant for billing and subscription data
- Is manageable by a two-person engineering team without dedicated DBA expertise
- Can handle 500 concurrent connections at launch, scaling to ~5,000 within 18 months

We evaluated three approaches to multi-tenancy: single database with tenant_id
column filtering, separate schema per tenant in one PostgreSQL cluster, and
separate database instance per tenant.

## Decision

Use PostgreSQL 16 with a separate schema per tenant within a single RDS
Multi-AZ instance. All tenant schemas are structurally identical, managed
by a single migration tool (Flyway) that applies migrations across all schemas.

## Rationale

Schema-per-tenant provides strong data isolation without the operational
overhead of managing thousands of database instances. Compared to row-level
filtering (tenant_id on every table), schema isolation means a missing WHERE
clause cannot accidentally expose another tenant's data — the schemas are
literally separate namespaces. Compared to separate databases, single-cluster
schema isolation allows connection pooling across tenants and simplifies backup
and monitoring.

Row-level filtering was rejected because it relies on application-layer
enforcement. A single missing WHERE clause leaks cross-tenant data. This is
unacceptable for a SOC 2 workload.

Separate databases were rejected because at 500+ tenants the operational
overhead (separate backup jobs, separate monitoring endpoints, separate
connection pools) outweighs the isolation benefits. We can migrate to
separate databases for large enterprise tenants if contractually required.

## Alternatives Considered

1. **Row-level security with tenant_id filtering**
   - Pros: Simpler migration tooling; no schema proliferation
   - Cons: Application-layer enforcement; one missed WHERE clause = data leak
   - Rejected: Unacceptable compliance risk for SOC 2

2. **Separate PostgreSQL instance per tenant**
   - Pros: Strongest isolation; easy to offer dedicated infra to enterprise customers
   - Cons: 500+ RDS instances; separate monitoring, backups, patching per instance
   - Rejected: Operationally infeasible for a two-person team at current scale

3. **MySQL / MariaDB**
   - Pros: Slightly faster for simple reads at high concurrency
   - Cons: Weaker JSON support; no schema-per-tenant pattern as well established;
     team expertise is PostgreSQL
   - Rejected: Marginal performance gain does not justify switching costs

## Consequences

Positive:
- Strong data isolation — schema boundaries enforced at database level
- Single migration tool applies to all tenants simultaneously
- Connection pooling works across all tenants
- Standard PostgreSQL tooling (pgAdmin, pgAnalyze, RDS native monitoring)

Negative:
- Schema proliferation: at 10,000 tenants, 10,000 schemas require careful
  catalog management (pg_catalog queries slow with large schema counts)
- Migration time grows linearly with tenant count — at 1,000 tenants a
  migration touching 50 tables takes ~5 minutes; plan for async migrations
- Cross-tenant analytics queries require UNION across schemas or a separate
  analytics database

Risks:
- If a large enterprise customer requires a dedicated PostgreSQL instance for
  contractual reasons, schema-per-tenant cannot accommodate this; we would need
  to migrate that tenant to a separate cluster

## Review Date: 2027-01-01
```

---

## Example 2: Auth Provider (Good ADR)

```markdown
---
adr_id: ADR-003
title: Delegate authentication to Auth0; own authorisation internally
date: 2026-04-15
status: accepted
---

## Context

We need authentication (verifying who a user is) and authorisation (what they
are allowed to do). Rolling our own auth has been the source of the most
security incidents in our previous products. We have a two-person engineering
team. The product will need SSO, MFA, and social login from day one for the
enterprise tier.

## Decision

Delegate authentication entirely to Auth0 (universal login, token issuance,
MFA, SSO via SAML/OIDC). Own authorisation internally using a Role-Based
Access Control (RBAC) model stored in our database, enforced at the API layer.

## Rationale

Auth0 has SOC 2 Type II certification, handles MFA and SSO integrations we
would otherwise spend months building, and shifts breach liability for
credential storage off our team. The $X/month cost at our scale is less than
the engineering time to build and maintain equivalent functionality.

We deliberately do NOT delegate authorisation to Auth0. Auth0's RBAC is
opinionated and tightly coupled to their permission model. Our permission
model will evolve with our product; owning it internally gives us full
control without Auth0 API rate limits on permission checks.

## Consequences

Positive:
- SSO (SAML + OIDC), MFA, social login available immediately
- Auth0 handles credential storage, breach notification, compliance
- JWT tokens from Auth0 are verified locally — zero latency on auth checks

Negative:
- Auth0 becomes a dependency for user login; Auth0 outage = users cannot log in
- Auth0 pricing scales with Monthly Active Users; model this at 100k MAU
- Permission model lives in two places (Auth0 user metadata + our DB);
  must stay in sync

## Review Date: 2026-10-15
```

---

## Anti-Patterns to Avoid

**Anti-pattern 1: Context-free decision**
```
❌ Bad:
## Context
We needed a caching layer.

## Decision
Use Redis.
```
This tells future engineers nothing about why Redis was chosen or what
problem it solved. In 18 months nobody will know whether to keep it.

**Anti-pattern 2: No alternatives**
```
❌ Bad:
## Alternatives Considered
None — Redis was the obvious choice.
```
If it was obvious, explain why. "Obvious" alternatives that weren't evaluated
are the source of "why did we do it this way?" conversations at 2am.

**Anti-pattern 3: No negative consequences**
```
❌ Bad:
## Consequences
- Fast caching layer ✓
- Easy to scale ✓
- Great community support ✓
```
Every decision has a cost. Redis adds operational complexity, requires
memory sizing, and needs a persistence strategy. State them.

**Anti-pattern 4: No review date**
An ADR without a review date is a decision that will never be revisited.
Technology changes. Constraints change. Teams change. Set the date.

**Anti-pattern 5: Documenting implementation, not the decision**
```
❌ Bad title: "Implemented Redis cache with 5-minute TTL for user sessions"
✅ Good title: "Use Redis as the session store instead of database-backed sessions"
```
The ADR records the decision and its rationale. The implementation details
live in EL-001 and the codebase.

---

## ADR Status Lifecycle

| Status | Meaning |
|--------|---------|
| `proposed` | Under discussion; not yet binding |
| `accepted` | Active decision; currently in effect |
| `deprecated` | The decision no longer applies but the record is preserved |
| `superseded` | Replaced by a newer ADR; link to the superseding ADR |

When a decision is reversed, do not delete or edit the original ADR. Create
a new ADR with `supersedes: ADR-NNN` and set the old ADR status to `superseded`.
The old ADR is a historical record — the reasons it was made, and the reasons
it was eventually changed, are both valuable.
