# CDR Escalation Templates

Use these templates when escalating to human operators. Fill in the bracketed
fields. Be precise — vague escalations waste human attention and delay resolution.

---

## Template 1: Unresolvable Conflict (CDR-004 trigger)

```
ESCALATION — CDR Conflict: Requires Human Decision
Goal: [G-YYYY-NNN] — [goal_statement]
Conflict ID: CDR-004-[NNN]
Detected: [ISO8601]

WHAT CONFLICTS:
[Precise statement of contradiction. E.g.: "PM-003 scopes SSO to SAML only.
SA-001 designs for both SAML and OIDC, adding 3 weeks to the EL estimate."]

AGENT POSITIONS:
  PM says: [exact claim from artifact, with artifact_id]
  SA says: [exact claim from artifact, with artifact_id]

CDR RECOMMENDATION:
[Specific recommended resolution with rationale. Do not say "up to you" —
take a position, even if the human may override it.]

CONSEQUENCE OF EACH OPTION:
  If PM's scope wins: [impact on timeline, cost, and downstream agents]
  If SA's design wins: [impact on timeline, cost, and downstream agents]

TASKS CURRENTLY BLOCKED:
  [List of task IDs and their agents that cannot proceed until this is resolved]

RESPONSE NEEDED BY: [deadline — typically 4 hours for P1, 1 hour for P0]
```

---

## Template 2: Bad DAG / Decomposition Failure (CDR-005 trigger)

```
ESCALATION — CDR Decomposition Audit: Bad DAG Detected
Goal: [G-YYYY-NNN] — [goal_statement]
Audit ID: CDR-005-[NNN]
Detected: [ISO8601]

WHAT FAILED:
[Precise description. E.g.: "Cycle detected: T3 depends on T4; T4 depends on T3.
Both tasks are owned by different agents and each claims the other's output as
a prerequisite."]

ROOT CAUSE ANALYSIS:
[Why did this happen? E.g.: "Goal statement was ambiguous — 'build and test'
conflated two tasks. Insufficient context about which agent owns the API contract."]

WHAT CDR TRIED:
[List resolution attempts already made. Be honest.]

WHAT IS NEEDED FROM HUMAN:
[Specific question or decision needed. Not "clarify the goal" — ask the exact
question. E.g.: "Does PM or SA own the API response schema definition?"]

RE-SUBMISSION PLAN:
[Once human provides input, CDR will: (1) ... (2) ... and re-submit CDR-001
within [time estimate]]
```

---

## Template 3: Cost Threshold Exceeded

```
ESCALATION — CDR Cost Alert: Threshold Exceeded
Goal: [G-YYYY-NNN] — [goal_statement]
Detected: [ISO8601]

CURRENT STATUS:
  Tasks completed: [N of M]
  Cost to date: $[actual]
  Cost estimated in CDR-001: $[estimate]
  Overrun: [%]

CAUSE OF OVERRUN:
[Specific task(s) responsible. Include: task ID, agent, expected cost, actual cost,
and why it ran over. E.g.: "T3 (EL — implementation) ran 3× estimate because
the SA-001 architecture required three additional service integrations not
anticipated at decomposition time."]

OPTIONS:
  Option A: Continue as planned — projected final cost: $[X]
  Option B: Descope [specific tasks] — saves $[Y], impacts [deliverables]
  Option C: Pause all non-critical tasks — review in [timeframe]

CDR RECOMMENDATION: [specific option with rationale]

RESPONSE NEEDED BY: [1 hour for P1 cost overrun; 4 hours for P2]
```

---

## Template 4: Agent Activation Request (Extended Team)

```
ESCALATION — CDR Activation Request: [AGENT_ID]
Goal: [G-YYYY-NNN] — [goal_statement]
Requested: [ISO8601]

AGENT TO ACTIVATE: [AGENT_ID — full name]
ACTIVATION TRIGGER (from spec): [quote the trigger condition from the spec]
EVIDENCE TRIGGER IS MET: [specific data point. E.g.: "First enterprise prospect
confirmed: Acme Corp, ACV $120k, in SAE pipeline since [date]"]

WHY NEEDED FOR THIS GOAL:
[Specific tasks in the current DAG that require this agent. Reference task IDs.]

IMPACT OF NOT ACTIVATING:
[What happens if human says no? Which tasks get reassigned or deferred?]
```

---

## Template 5: Human Override Acknowledgement (RM or CPO veto overridden)

```
ESCALATION ACKNOWLEDGEMENT — Override Logged
Veto type: [RM deployment veto | CPO compliance veto]
Override requested by: [human operator name/ID]
Goal: [G-YYYY-NNN]
Overridden artifact: [RM-001-xxx or CPO-003-xxx]

REASON STATED BY HUMAN OPERATOR:
[Verbatim or close paraphrase of business justification given]

RISKS ACCEPTED:
[Itemised list of risks that the veto was designed to prevent, now accepted by
the human operator's decision]

AUD NOTIFICATION: Sent at [ISO8601]
LOG ENTRY: CDR-004-[NNN] created with full record

CDR NOTE: CDR does not endorse this override. The veto existed for documented
reasons. This log entry is a record of human authority, not CDR agreement.
```
