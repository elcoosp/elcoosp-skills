# Messaging Framework — Positioning Canvas and Copy Frameworks

MK uses this framework when writing positioning in MK-001 (GTM Plan) and
producing copy for campaigns, landing pages, and sales enablement.

---

## The Positioning Canvas

Fill this in before writing any copy. Every piece of copy is downstream of
this canvas. If the canvas is vague, the copy will be vague.

```
FOR:        [specific persona — not "companies" but "VP of Engineering at a
             50–500 person SaaS company"]

WHO:        [their specific problem — one sentence, in their language]

OUR PRODUCT [product name]

IS A:       [category — what kind of thing is it]

THAT:       [primary benefit — what problem does it solve]

UNLIKE:     [primary alternative — what they're doing today instead]

OUR PRODUCT [key differentiator — why us, not the alternative]
```

**Example (good):**
```
FOR:        VP of Engineering at a 50–500 person B2B SaaS company

WHO:        spend 20% of their week on manual handoffs between product, design,
            and engineering, creating delays in every feature cycle

OUR PRODUCT FeatureFlow

IS A:       product development coordination platform

THAT:       eliminates the PRD-to-ticket translation layer by connecting your
            product context directly to your engineering workflow

UNLIKE:     Notion docs + Jira tickets + Slack threads that fragment context
            across tools and require constant manual sync

OUR PRODUCT generates structured engineering tickets directly from product
            requirements with full traceability, cutting feature cycle time
            by 40%
```

**Example (bad — too vague to be useful):**
```
FOR:        software teams
WHO:        struggle with communication
OUR PRODUCT TeamSync
IS A:       collaboration tool
THAT:       improves productivity
UNLIKE:     other tools
OUR PRODUCT is better and easier to use
```

The bad example could describe 500 products. It will produce copy that sounds like every other SaaS homepage.

---

## Messaging Hierarchy

Once the positioning canvas is complete, translate it into a messaging hierarchy.
Every piece of copy (landing page, ad, email) uses a subset of this hierarchy.

```
Level 1 — Headline (10 words max)
  What you do for whom, stated as an outcome
  Test: can a stranger read this and know what you do and for whom?

Level 2 — Sub-headline (25 words max)
  How you do it differently from the alternative
  Test: does this differentiate, or could a competitor say the same thing?

Level 3 — Proof points (3 maximum)
  Specific, verifiable claims — not "powerful" or "seamless"
  Format: [metric] for [customer] using [feature]
  Test: can this be verified? Would a skeptic accept it?

Level 4 — Objection handling
  The top 3 reasons prospects don't buy, pre-empted
  (from SAE-005 market intelligence notes)

Level 5 — CTA (1 primary action)
  One next step; specific and low-friction
  Not "Learn more" — "See a 5-minute demo" or "Start free for 14 days"
```

---

## Forbidden Words

These words appear on every B2B SaaS homepage and carry zero differentiation:

| Forbidden | Why | Replace with |
|-----------|-----|-------------|
| Powerful | Says nothing | Describe the specific power |
| Seamless | Overused; unverifiable | Describe the specific friction eliminated |
| Intuitive | Everyone claims it | Show, don't tell — use a screenshot |
| Best-in-class | Unverifiable superlative | Cite the specific metric or customer |
| Next-generation | Meaningless | Describe what's actually new |
| Robust | Vague | Specify what it handles that competitors don't |
| Scalable | Expected, not differentiating | Give the scale numbers |
| Easy to use | Everyone says it | Describe the time-to-value in minutes |
| Streamlined | Overused | Describe the specific steps eliminated |
| Comprehensive | Vague | Name the specific capabilities |

---

## Copy Frameworks for Different Contexts

### Problem-Agitate-Solution (PAS) — for cold outbound and ads

```
Problem:  [State the specific problem the persona has]
Agitate:  [Describe the cost/pain of not solving it — make it vivid]
Solution: [Introduce the product as the resolution]
```

**Example:**
```
Every sprint, your engineers spend the first day re-reading product docs
and asking PMs for clarifications before writing a single line of code.

At 50 engineers, that's 50 days of engineering time per sprint — going
backwards instead of forwards.

FeatureFlow connects product requirements directly to your engineering
workflow. Engineers start with full context. No re-reading. No Slack threads.
Just work.
```

### Before-After-Bridge (BAB) — for landing pages and demos

```
Before: [Current painful state]
After:  [Future desired state]
Bridge: [How to get there — the product]
```

**Example:**
```
Before: Feature requests live in Notion, specs in Google Docs, tickets in Jira,
        and context in Slack. Finding the "why" behind any ticket takes 20 minutes.

After:  Every ticket links to its requirement, its design rationale, and its
        acceptance criteria. New engineers ramp in days, not months.

Bridge: FeatureFlow creates this connected context layer automatically —
        no migration, no new process, no change to how your team works.
```

### Jobs-To-Be-Done framing — for product positioning and sales discovery

JTBD: customers don't buy products; they hire them to do a job they couldn't
do before (or couldn't do well enough).

The job has three dimensions:
- **Functional:** The practical task ("I want to reduce time spent on handoffs")
- **Emotional:** How they want to feel ("I want to feel confident the team is aligned")
- **Social:** How they want to be perceived ("I want to be the PM who ships on time")

Copy that addresses all three is more persuasive than copy that addresses only the functional.

---

## Value Proposition Testing Checklist

Before using any value proposition in a campaign:

- [ ] Is it specific? (Could it describe a competitor's product? If yes, make it more specific)
- [ ] Is it verifiable? (Can a customer check this claim? If not, remove it or add evidence)
- [ ] Is it relevant? (Does it address a problem in PM-001's problem statements? If not, revisit)
- [ ] Is it differentiated? (Does the primary alternative offer the same? If yes, go deeper)
- [ ] Is the language the customer's? (Pulled from user interviews in PM-001, not from inside the company)
- [ ] Does it pass the "So what?" test? (Read the claim aloud; ask "so what?" — if you can, add the answer)
