# Queue State Schema — Agent State Store

SCH persists all task queue state to the Agent State Store so it can
reconstruct full state after any restart. This file defines the key-value
schema for every piece of state SCH writes.

---

## Key Namespace

All SCH keys follow the pattern: `sch:{type}:{identifier}`

---

## Task Queue (per goal)

**Key:** `sch:queue:{goal_id}`
**Value:** JSON array of task queue entries

```json
[
  {
    "task_id": "T1",
    "goal_id": "G-2026-004",
    "agent_id": "PM",
    "capability_tier": "reasoning_balanced",
    "resolved_model": "claude-sonnet-4-6",
    "status": "completed",
    "dependencies": [],
    "dependencies_met": true,
    "output_artifact_id": "PM-003-sso-prd-v1",
    "estimated_effort_hours": 6,
    "estimated_cost_usd": 0.54,
    "actual_cost_usd": 0.61,
    "tokens_used": 203333,
    "retry_count": 0,
    "first_queued_at": "2026-04-15T08:00:00Z",
    "dispatched_at": "2026-04-15T08:01:00Z",
    "completed_at": "2026-04-15T14:30:00Z",
    "last_error": null
  }
]
```

**Task status values:**

| Status | Meaning |
|--------|---------|
| `waiting` | Dependencies not yet met |
| `ready` | All dependencies met; awaiting dispatch slot |
| `dispatched` | Task sent to agent; awaiting completion |
| `retrying` | Failed; within retry budget; waiting for backoff |
| `completed` | Output artifact is active in Artifact Store |
| `failed_exhausted` | 3 retries exhausted; escalated to CDR |
| `cancelled` | CDR cancelled this task (DAG re-plan) |
| `blocked_quarantine` | Dependency artifact quarantined by AUD |

---

## Goal Cost Accumulator

**Key:** `sch:cost:{goal_id}`
**Value:**

```json
{
  "goal_id": "G-2026-004",
  "estimated_cost_usd": 28.40,
  "actual_cost_usd_to_date": 12.34,
  "tasks_completed": 3,
  "tasks_total": 6,
  "last_updated": "2026-04-15T14:30:00Z",
  "threshold_50pct_alerted": false,
  "threshold_150pct_alerted": false,
  "threshold_200pct_paused": false
}
```

---

## Model Health State

**Key:** `sch:model-health:{tier}`
**Value:**

```json
{
  "tier": "reasoning_heavy",
  "primary_model": "claude-opus-4-6",
  "fallback_model": "claude-sonnet-4-6",
  "primary_consecutive_failures": 0,
  "last_primary_failure_at": null,
  "fallback_active": false,
  "fallback_activated_at": null,
  "primary_recovery_count": 0,
  "last_updated": "2026-04-15T14:30:00Z"
}
```

---

## Rate Limit Tracking

**Key:** `sch:rate-limit:{model_name}`
**Value:**

```json
{
  "model": "claude-sonnet-4-6",
  "tier": "reasoning_balanced",
  "limit_tokens_per_minute": 80000,
  "tokens_used_current_minute": 12450,
  "window_start": "2026-04-15T14:30:00Z",
  "window_end": "2026-04-15T14:31:00Z",
  "dispatches_paused": false,
  "paused_since": null
}
```

SCH resets `tokens_used_current_minute` at the start of each new minute window.
If `tokens_used_current_minute` exceeds 80% of the limit, SCH slows dispatch
(200ms inter-dispatch delay). At 95%, SCH pauses dispatch until the next window.

---

## Active Goals Registry

**Key:** `sch:active-goals`
**Value:** JSON array of goal IDs currently being executed

```json
["G-2026-004", "G-2026-005"]
```

SCH adds a goal ID when it ingests the CDR-001. It removes the goal ID when
it emits `goal_completed` or `goal_blocked`. Max concurrent active goals: 5
(configurable). If a 6th goal arrives, SCH queues it and notifies CDR.

---

## Execution Metrics Log (append-only)

**Key pattern:** `sch:metrics:{goal_id}:{task_id}`
**Value:**

```json
{
  "goal_id": "G-2026-004",
  "task_id": "T1",
  "agent_id": "PM",
  "model": "claude-sonnet-4-6",
  "tier": "reasoning_balanced",
  "dispatched_at": "2026-04-15T08:01:00Z",
  "completed_at": "2026-04-15T14:30:00Z",
  "duration_seconds": 22740,
  "tokens_input": 1842,
  "tokens_output": 3211,
  "tokens_total": 5053,
  "cost_usd": 0.0152,
  "retry_count": 0,
  "timeout_occurred": false,
  "fallback_used": false
}
```

SCH writes this record on task completion and sends a copy to AUD via the
Event Bus (`execution_metric` event type). AUD uses these for cost runaway
detection and hallucination pattern analysis.
