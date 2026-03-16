# Lifecycle Email Templates by Trigger

CSD uses these templates when building automated journeys in CSD-001 and
CSD-002. Every template requires: subject line, preview text, body, CTA,
and the suppression condition that stops the sequence.

All templates must be reviewed against the active product (EL-001) before
deploying — never describe features that don't exist.

---

## Template Set 1: Onboarding Sequence (Days 1–14)

### Email 1 — Welcome (Trigger: signup confirmed)

```
Subject:    You're in — here's where to start
Preview:    One thing to do in the next 10 minutes.

Body:
Hi [first_name],

Welcome to [product]. You're all set.

The fastest way to get value is [single most important first action — 
from PM-001 time-to-first-value definition].

[Primary CTA button: "Do the thing" — specific label]

Takes about 10 minutes. We'll check in tomorrow.

[Name]
[Product]

---
[Unsubscribe] | [Preferences]
```

**Design notes:**
- One action. One button. No secondary links until Day 3.
- "Check in tomorrow" sets the expectation without being creepy.
- Subject line is a statement, not a question. Questions feel like homework.

**Suppression:** Stop this sequence if `first_value_event` fires.

---

### Email 2 — Day 3: Progress check (Trigger: 3 days since signup, first_value_event NOT fired)

```
Subject:    Did you get stuck somewhere?
Preview:    Takes 2 minutes to find out.

Body:
Hi [first_name],

You signed up 3 days ago but haven't [completed first_value_event yet].

That usually means one of three things:
- Something in the setup was confusing
- You got pulled away and forgot
- [Product] isn't the right fit (that's okay too)

Which is it?

[Button: "I got stuck — show me how"]
[Button: "I'll do it now"]
[Button: "This isn't for me"]

Whatever you pick, I'll make it easy.

[Name]
```

**Design notes:**
- Acknowledge the reality (they didn't do the thing). Don't pretend it didn't happen.
- Give them three honest options including the opt-out. This builds trust.
- The "I got stuck" click routes to a specific help article or triggers a support ping.

**Suppression:** Stop if `first_value_event` fires.

---

### Email 3 — Day 7: Feature highlight (Trigger: 7 days since signup, first_value_event fired)

```
Subject:    Now that you've [completed first action], try this
Preview:    Most users do this next.

Body:
Hi [first_name],

Nice — you've [completed first action]. That's the foundation.

The next thing most [persona] do is [second key action]. It takes about
15 minutes and [specific benefit].

[1-sentence explanation of the feature, in plain language]

[Button: "Try [second action]"]

Or if you have questions, [link to docs / chat].

[Name]
```

**Suppression:** Stop if `second_key_event` fires or user is marked churned.

---

## Template Set 2: Feature Adoption (Trigger: Core feature unused after 14 days)

```
Subject:    You haven't used [feature name] yet
Preview:    Here's why it matters.

Body:
Hi [first_name],

One thing I've noticed: you haven't tried [feature name] yet.

Most [product] users find this is the part that saves them the most time,
specifically because [single concrete benefit — one sentence].

Here's a 2-minute overview:
[Link to short video or interactive demo]

Or just [primary action] and see for yourself.

[Button: "Try [feature name]"]

[Name]

P.S. If this isn't relevant to how you use [product], just let me know
what you're focused on and I'll tailor what I send you.
```

**Notes:**
- "You haven't used X yet" is direct. Don't soften it into something vague.
- The P.S. is important — it gives people a graceful way to update their preferences.

---

## Template Set 3: Usage Drop (Trigger: Weekly active usage drops ≥ 50% vs prior 4-week average)

```
Subject:    Everything okay at your end?
Preview:    We noticed a drop in activity.

Body:
Hi [first_name],

I noticed [product] activity dropped off over the last couple of weeks.

Sometimes that means a busy period, sometimes it means something's
not working the way it should.

If you've hit a snag or something changed in your workflow, I'm happy
to help figure it out. Just hit reply.

If everything's fine and you've just been heads-down, ignore this —
we'll be here when you're ready.

[Name]
```

**Notes:**
- Short and human. No marketing. No product pitch. Just a check-in.
- "Just hit reply" makes the response path frictionless.
- Do not include a product feature CTA in this email. It reads as opportunistic.

**Suppression:** Do not send if CSH is managing this account. Check CRM ownership.

---

## Template Set 4: Pre-Renewal (Trigger: 60 days before subscription renewal)

```
Subject:    Your [product] subscription renews in 60 days
Preview:    Here's what you've accomplished.

Body:
Hi [first_name],

Your [product] subscription renews on [date]. No action needed —
it'll renew automatically.

Before that happens, I wanted to share what your team has done
with [product] this year:

[Personalised usage stat 1: e.g., "X workflows created"]
[Personalised usage stat 2: e.g., "Y hours saved"]
[Personalised usage stat 3: e.g., "Z team members active"]

[If usage is strong: "You're getting great value. If you'd like to
explore an annual plan at a discount, reply and I'll send details."]

[If usage is weak: "We want to make sure [product] is working for you.
If there's anything we can help with before renewal, reply and we'll
set up a call."]

[Name]
```

**Notes:**
- Use real data. Personalise with actual stats from the product analytics integration.
- The "strong usage" branch tries for annual upsell; the "weak usage" branch is retention.
- CSD routes weak-usage pre-renewal accounts to CSH if they're enterprise-sized.

---

## Template Set 5: Win-Back (Trigger: 30 days after cancellation)

```
Subject:    [first_name], what could we have done differently?
Preview:    A quick question, no sales pitch.

Body:
Hi [first_name],

You cancelled [product] about a month ago. I'm not writing to try to
win you back — I'm writing because I genuinely want to understand
what didn't work.

If you have 2 minutes, would you mind telling me?

[Button: "Tell us what happened" → short survey]

If things have changed and you want to give [product] another look,
we'll make sure the experience is better. But no pressure.

[Name]
```

**Notes:**
- One email only. Do not send a win-back sequence. One honest ask.
- Suppress permanently after one send regardless of response.
- Route survey responses to PM as market intelligence.
