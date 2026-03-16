# Shared Artifact Conventions (v4.0)

All agents in the Virtual SaaS Team produce artifacts that conform to this spec.
Read this file whenever producing or consuming any artifact.

---

## Universal Artifact Header

Every artifact **must** begin with this YAML frontmatter block. AUD will reject any
artifact missing required fields.

```yaml
---
artifact_id: {AGENT_ID}-{SEQ}-{slug}-v{N}       # e.g. PM-003-sso-prd-v1
agent: {AGENT_ID}
version: {N}
status: draft                                      # draft → pending_review → active
schema_version: 4.0
goal_id: {G-YYYY-NNN}
task_id: {TN}
created: {ISO8601}
author_agent: {AGENT_ID}
upstream_artifacts:
  - {artifact_id}: {relationship}                  # e.g. PM-001: informed_by
downstream_consumers:
  - agent: {AGENT_ID}
    expects: {artifact_id or description}
checksum_sha256: {computed on content below header}
---
```

## Artifact Naming Convention

```
{AGENT_ID}-{SEQ}-{slug}-v{VERSION}.{FORMAT}

Examples:
  PM-003-sso-prd-v1.md
  SA-001-sso-architecture-v2.yaml
  EL-004-api-implementation-v1.zip
  QA-001-sso-test-plan-v1.md
```

## Artifact Lifecycle

```
draft → pending_review → active → superseded (archived, never deleted)
                       ↘ quarantined (AUD flag; awaits human review)
```

- **Never delete** an artifact. Supersede it by producing a new version.
- **Quarantined** artifacts cannot be consumed by downstream agents until a human clears them.

## Format by Artifact Type

| Type | Format | Validation |
|------|--------|------------|
| Requirements, PRDs, specs | Markdown + YAML frontmatter | Required sections present |
| Architecture diagrams | Mermaid in Markdown | Parseable |
| Data schemas | YAML or JSON Schema | JSON Schema Draft 7 |
| Test results | JUnit XML or JSON | Standard report format |
| API specs | OpenAPI 3.1 YAML | openapi-validator |
| Code deliverables | Language files in ZIP | Passes linter + tests |
| Configuration | YAML | Schema valid |

## Cross-Reference Rules

1. Every artifact lists its **upstream_artifacts** (what it consumed).
2. Every artifact lists its **downstream_consumers** (who to notify).
3. Orphaned artifacts (no upstream + no downstream) generate an AUD warning.
4. Factual quantitative claims must cite a source artifact — no unsourced statistics.

## Capability Tiers (never hardcode model names)

| Tier | Use for |
|------|---------|
| `reasoning_heavy` | Architecture, compliance, complex strategy |
| `reasoning_balanced` | PRDs, implementation, analysis |
| `fast` | Routing, test execution, content generation |
| `deterministic` | Rule-based logic; no LLM |

Model names are resolved at runtime from `/config/model-registry.yaml`.

## Escalation on Blocking Issues

```
1. Log blocker event → Event Bus
2. CDR notified (15-min resolution window)
3. If unresolved → human operator (P0: 15min, P1: 1hr SLA)
4. SCH suspends dependent tasks pending resolution
```
