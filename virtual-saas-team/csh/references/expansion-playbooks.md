# Expansion Playbooks — Proven Motion Templates

CSH uses these playbooks when identifying and executing expansion opportunities.
Expansion is the engine of NRR > 100%. A customer who grows their investment
is the strongest signal the product is delivering value.

---

## The Expansion Mindset

Expansion is not upselling. Upselling is pushing customers to buy more than
they need. Expansion is recognising when a customer's needs have grown beyond
what they currently have — and making it easy for them to grow with the product.

The question is never "how do I get them to spend more?" It is "what would
they need to do more, and how does our product support that?"

---

## Playbook 1: Seat Expansion

**Trigger signals:**
- Usage approaching or at the seat limit
- Champion mentions they want to roll out to a new team
- New hire in champion's team who needs access
- Champion is added to Slack and mentions onboarding a colleague

**Timing:** Bring up expansion when the customer is at ~80% seat utilisation —
not at 100% (frustrated) and not at 40% (no urgency).

**Conversation:**
```
"I noticed your team is getting close to the seat limit. Before that
becomes a constraint, I wanted to check in — are there other teams at
[company] who've expressed interest in using [product]?

[If yes:] Great. What would need to happen to bring them on? I can
put together some numbers for you.

[If no:] Good to know. Just wanted to make sure you had room to grow
without hitting a wall."
```

**Motion:**
1. CSH identifies accounts at ≥ 80% seat utilisation
2. CSH reaches out proactively (not waiting for the customer to ask)
3. If expansion confirmed: CSH introduces SAE for commercial negotiation
4. CSH stays involved through the expansion onboarding

**Do not:** Surprise a customer with a limit-hit error. They should know
the limit is approaching before it matters.

---

## Playbook 2: Tier Upgrade (Feature-Led)

**Trigger signals:**
- Customer repeatedly asks about a feature that's in the next tier
- Customer is using a workaround for something the next tier handles natively
- Customer's team size / usage volume has crossed the tier threshold
- QBR reveals a stated goal that requires a higher-tier capability

**Conversation:**
```
"You mentioned last quarter that [their stated goal] was a priority. 
I've been thinking about that — [higher tier feature] was actually
built exactly for that use case. 

I could show you a quick demo of how [similar company] used it to 
[specific outcome]. Would that be worth 20 minutes?"
```

**Motion:**
1. CSH maps stated goals (from CSH-001 success plan) to product capabilities
2. When a goal maps to a feature above their current tier: schedule a
   targeted mini-demo (not a full product tour — 15 minutes max)
3. If the customer is interested: CSH loops in SAE and CPQ for commercial terms
4. CSH monitors adoption of the new feature post-upgrade

**Anti-pattern:** Pitching a tier upgrade during or right after a support
escalation. Timing matters. Wait for a positive moment.

---

## Playbook 3: Annual to Multi-Year Renewal

**Trigger signals:**
- Renewal approaching (90-day window)
- Health score Green (75+)
- Champion is stable and engaged
- Customer has measurable success story to reference

**Timing:** Start the multi-year conversation at 90 days before renewal.
At 30 days it feels like a last-minute push. At 90 days it feels like
strategic planning.

**Conversation:**
```
"We're coming up on your renewal in [N] months, and I wanted to have
a strategic conversation before we get into the commercial discussion.

Looking back at the year, [specific outcome they achieved]. 
What's on your roadmap for next year that you'd want [product] to support?

[After they share:]

A lot of our customers in [their situation] have moved to a 2- or 3-year
arrangement at this point — it gives them better pricing and locks in
the roadmap investment we make in their success. Is that something worth
exploring for you?"
```

**Motion:**
1. CSH identifies renewal accounts 90 days out
2. CSH produces QBR (CSH-003) showing year-in-review value delivered
3. QBR sets up the multi-year conversation naturally
4. SAE handles commercial negotiation; CSH remains the relationship owner

**Discount guidance:** Multi-year discounts are CPQ's domain. CSH should
not quote discounts without CPQ input. Typical: 10–15% for 2-year, 20–25%
for 3-year — but CPQ sets the actual number based on margin and deal size.

---

## Playbook 4: New Product / Add-On

**Trigger signals:**
- Customer asks about something your product doesn't do yet → check if a
  new add-on solves it
- MK launches a new feature or add-on in MK-001 that maps to a customer pain
- Champion changes roles and new role has different needs

**Conversation:**
```
"We just launched [add-on] that I thought of immediately when I heard
you mention [their pain] in our last call.

It's specifically built for [use case]. I'd love to show you a 
15-minute overview and see if it fits what you described. 
When do you have a few minutes this week?"
```

**Motion:**
1. MK notifies CSH when new features or add-ons ship (via MK-001 → Event Bus)
2. CSH filters customer base for accounts with matching pain signals
3. CSH does targeted outreach (not mass email — personalised based on CSH-001)
4. 15-minute focused demo → SAE/CPQ if commercial discussion needed

---

## Tracking Expansion Performance

CSH tracks expansion in CPQ (as SAE-driven expansion opportunities) or
directly as CSH-generated revenue. The metric is Expansion MRR.

Monthly CSH expansion report:
```
expansion_mqls_generated: accounts CSH identified as expansion candidates
expansion_mrr_closed: MRR added from CSH-originated expansion
expansion_pipeline: current value of open expansion opportunities
nrr_contribution: expansion MRR / opening MRR × 100
top_expansion_playbook: which playbook generated most MRR this month
```

Target: CSH-originated expansion should represent at least 60% of total
expansion MRR. The rest comes from SAE-identified upsells. If the split
is reversed (SAE > CSH on expansion), customers aren't getting proactive
enough engagement.
