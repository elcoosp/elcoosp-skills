# Documentation Templates by Type

TW uses these templates when producing new documentation artifacts.
Each template defines the required sections, the questions each section
must answer, and the source artifacts to pull from.

---

## Template 1: Quickstart Guide (TW-002)

**Purpose:** Get a new user to their first meaningful outcome in < 15 minutes.
**Source artifacts:** EL-001 (Implementation Summary), PM-001 (Product Context Doc)
**Target reader:** New user, first session with the product.

```markdown
# [Product/Feature name]: Quickstart

**Time:** [N] minutes
**What you'll accomplish:** [Specific outcome in one sentence]

## Before you begin

[List exactly what the reader needs — software installed, credentials ready, etc.
Be specific. If they need Node.js 18+, say "Node.js 18 or higher (check with `node --version`)"]

## Step 1: [Action verb + object]

[Instruction — one action per step]

```[code if applicable]```

You should see: [exact expected output or UI state — screenshot optional but helpful]

## Step 2: [Action verb + object]

[Instruction]

## Step 3: [Action verb + object]

[Instruction]

## You're done

[What the user has accomplished. What they can do with it now.]

## What's next

- [Next logical task with link]
- [Alternative next task with link]
- [Link to full reference docs]

## Troubleshooting

**[Error message or symptom]**
[Cause in plain language] → [Specific fix]

**[Another common error]**
[Cause] → [Fix]
```

**Checklist before publishing:**
- [ ] Tested every step myself on a clean environment
- [ ] Every code snippet runs without modification
- [ ] "You should see" sections match actual output
- [ ] Troubleshooting section covers at least 2 common failure points
- [ ] All technical facts sourced from EL-001 or SA-002

---

## Template 2: How-To Guide (TW-004)

**Purpose:** Help an existing user accomplish a specific task.
**Source artifacts:** EL-001, SA-002 (API spec if relevant)
**Target reader:** User who knows the product and needs to do a specific thing.

```markdown
# How to [specific task]

[One sentence: what this guide covers and when you'd need it]

## Prerequisites

- [What the user needs before starting — be specific]
- [Existing configuration or permissions required]

## Steps

1. [First action — start with a verb]

   [Additional context if needed — keep brief]

2. [Second action]

   ```[code if applicable]```

3. [Third action]

## Result

[What the user has when they're done. Make it concrete.]

## Related guides

- [How to do adjacent task]
- [How to undo this]
```

**Rules:**
- One task per guide. If the guide covers two tasks, split it.
- Every step starts with an imperative verb.
- The result section tells them what they have, not what the product did.

---

## Template 3: API Reference Entry (TW-003)

**Purpose:** Complete technical reference for a single API endpoint.
**Source artifacts:** SA-002 (OpenAPI 3.1 spec) — this is the authoritative source.
TW transforms the machine-readable spec into human-readable documentation.

```markdown
## [HTTP Method] [Path]

[One sentence: what this endpoint does in plain language]
[One sentence: when you would use it]

**Authentication:** [Required / Not required] — [auth method]
**Rate limit:** [N requests per minute / hour]

### Request

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `param_name` | string | Yes | [What it is; valid values if constrained] |
| `param_name` | integer | No | [What it is; default value; range] |

#### Request body

```json
{
  "field_name": "example_value",
  "field_name": 42
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `field_name` | string | Yes | [Description; constraints] |

### Response

#### Success (200 OK)

```json
{
  "id": "res_abc123",
  "status": "active",
  "created_at": "2026-04-15T10:23:00Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier |
| `status` | string | One of: `active`, `inactive`, `pending` |

#### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `invalid_request` | [What causes this; how to fix it] |
| 401 | `unauthorized` | API key missing or invalid |
| 404 | `not_found` | Resource with that ID does not exist |
| 429 | `rate_limited` | [Rate limit exceeded; Retry-After header contains wait time] |

### Example

#### curl

```bash
curl -X POST https://api.product.com/v1/[path] \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"field": "value"}'
```

#### Response

```json
{
  "id": "res_abc123",
  "status": "active"
}
```
```

**Rules:**
- Every parameter must have a description — "The ID" is not a description.
- Every error status must include how to fix it, not just what it means.
- Code examples must work exactly as written. Test them.
- If the SA-002 spec and actual behaviour differ, raise with EL before publishing.

---

## Template 4: Troubleshooting Article

**Purpose:** Help a user who has hit a specific error or unexpected behaviour.
**Source artifacts:** EL-001 (known limitations section), QA-002 (bug reports with workarounds)

```markdown
# [Error message or symptom — quoted exactly as the user sees it]

## What this means

[Plain language explanation of why this happens. Not the technical root cause —
what's happening from the user's perspective.]

## How to fix it

[Numbered steps to resolution. Most common fix first.]

1. [First thing to try]
2. [Second thing to try if first didn't work]
3. [Third option for edge cases]

## If none of these work

[What to do next — link to support, provide information to include in a report]

## Related issues

- [Link to related error]
- [Link to related how-to guide]
```

**Rules:**
- Title is the exact error message the user sees — this is what they'll search for.
- "What this means" must not blame the user or be condescending.
- Always include a path to human support if the fixes don't work.
