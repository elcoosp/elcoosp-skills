# In-App Messaging Patterns and Best Practices

CSD uses these patterns when designing in-app engagement campaigns in CSD-001
and CSD-002. In-app messages have higher engagement than email because they
appear at the moment of intent — but they also have higher annoyance potential
if misused.

---

## The One-Per-Session Rule

Never show more than one unsolicited message per session. A session is
defined as a continuous period of user activity with no more than 30 minutes
of inactivity.

This rule is enforced in the campaign configuration, not left to individual
campaign designers. The messaging platform should have a global frequency cap
that prevents multiple campaigns from firing simultaneously.

---

## Pattern 1: Tooltip

**Use for:** Contextual help on a specific UI element. Appears on hover or
first encounter with a new feature.

**Trigger:** User hovers over element OR first time element is seen (one-time show).

**Content rules:**
- Maximum one sentence. If it needs more, the UI is unclear — fix the UI.
- Answer "what does this do?" not "how does this work?" (docs handle the latter)
- No CTA unless there's a clear next action

**When NOT to use:**
- For marketing a new feature (use Feature Announcement instead)
- For guidance that appears on every hover (use empty state or label copy)

**Template:**
```
[UI element label]
"[One sentence describing what this element does or what happens when you interact with it]"
```

**Example:**
```
Health Score
"A weighted score based on usage, feature adoption, and support activity.
Updates weekly."
```

---

## Pattern 2: Feature Announcement Banner

**Use for:** Notifying existing users of a new feature they can now use.
Appears at top of relevant page, one time.

**Trigger:** First login after the feature ships (synced with RM-002 release date).

**Suppression:** User dismisses it, or completes the feature's key action.

**Content rules:**
- Headline: what's new in 5 words or fewer
- Body: one sentence on what it does for them (not what it is)
- CTA: single action, specific label
- Dismiss: always available; permanent (never shows again after dismissed)

**Template:**
```
[Icon] New: [Feature name]
[One sentence: what this enables the user to do]
[CTA button: specific action]    [Dismiss ×]
```

**Example:**
```
✦ New: Bulk export
Download all your reports as a single CSV in one click.
[Export now]    [×]
```

---

## Pattern 3: Empty State Message

**Use for:** The moment before the user has any data in a section.
This is not a campaign — it's a permanent part of the UI designed by DX
and written by UE/CSD. Included here because CSD often writes the copy.

**Content rules:**
- Headline: an action, not a description. "Add your first contact" not "No contacts yet."
- Body: why this section matters and what unlocks when they act. 1–2 sentences max.
- CTA: primary action only

**Template:**
```
[Illustration or icon]
[Action-oriented headline]
[1-2 sentences: what this enables]
[Primary CTA button]
[Secondary: "Learn how" link to docs — optional]
```

**Example:**
```
[Icon: pipeline]
Connect your first integration
See all your customer data in one place — support tickets, product usage, and billing status.
[Connect an integration]
[How integrations work →]
```

---

## Pattern 4: Onboarding Checklist

**Use for:** Guiding users through a multi-step onboarding journey within the product.
Persistent, collapsible, dismissible. Shows progress.

**Trigger:** Activates on first login after signup. Disappears when all items are checked.

**Content rules:**
- Maximum 5 items. More than 5 overwhelms.
- Each item is a single action, not a concept
- Items are ordered by value-to-effort (easiest + highest-value first)
- Each item has a completion event that auto-checks the box

**Template:**
```
Get started with [product]
[progress bar: N of 5 complete]

✓ [Completed action — greyed out]
○ [Next action — highlighted, with "Do it" link]
○ [Future action]
○ [Future action]
○ [Future action]

[Dismiss — "I'll explore on my own"]
```

**Example:**
```
Get started with Pipeline
[■■□□□] 2 of 5 complete

✓ Add your first contact
✓ Connect your email
○ Create your first deal    [Create deal →]
○ Set up a pipeline stage
○ Invite a teammate
[I'll explore on my own]
```

---

## Pattern 5: Contextual Upgrade Prompt

**Use for:** When a user attempts an action that requires a higher tier.
Shows at the moment of friction — not before, not in a general campaign.

**Trigger:** User clicks a locked feature or hits a plan limit.

**Content rules:**
- Acknowledge what they were trying to do
- Explain the limitation briefly (not apologetically)
- Show the clear path to resolution
- Never block or punish — always give a graceful exit

**Template:**
```
[Feature name] is on the [tier] plan
[One sentence: what they'd get if they upgraded]
[CTA: "Upgrade to [tier]"]    [Learn more]
```

**Example:**
```
Advanced analytics is on the Pro plan
Unlock cohort analysis, custom dashboards, and CSV export for your whole team.
[Upgrade to Pro]    [See what's included]
```

---

## Frequency Cap Configuration

Configure these in your in-app messaging platform. These are maximums across
all campaigns — individual campaigns should have their own suppression rules
on top of these global caps.

```
Global caps:
  Unsolicited modals:  1 per session, max 3 per week
  Banners:             1 visible at a time; max 2 different banners per week
  Tooltips:            No cap (triggered by user action, not pushed)
  Checklists:          No cap (persistent, dismissible)
  Upgrade prompts:     Shown at moment of friction only; no frequency cap
```

---

## Suppression List — Always Check Before Sending

Before any in-app message fires, check:
- Is this account managed by CSH? (If yes, suppress — CSH handles their messaging)
- Has the user dismissed this campaign permanently?
- Has the user completed the target action? (If yes, suppress — don't show after they've done it)
- Is the user in their first 24 hours? (Consider suppressing all non-onboarding messages)
- Is the account currently experiencing a support incident? (Suppress promotional messages)

These checks run in the platform configuration, not manually per campaign.
