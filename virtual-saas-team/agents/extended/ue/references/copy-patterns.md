# In-Product Copy Patterns by Element Type

UE uses these patterns when writing copy for in-product educational elements
in UE-001 (onboarding flows) and UE-002 (adoption campaigns). Every element
type has its own copy constraints — follow them.

---

## The Core Principle

In-product copy competes directly with the product itself for attention.
The user is trying to accomplish something. Every word that doesn't help
them accomplish it is an obstacle.

Shorter always wins. Not because brevity is a style choice — because the
user is not here to read. They are here to do.

---

## Pattern 1: Modal (Onboarding)

**Use:** First encounter with a key workflow. Appears once.

**Word budget:**
- Headline: 6 words maximum
- Body: 25 words maximum
- CTA: 4 words maximum

**Formula:**
```
Headline: [What this does for them] — action-oriented, not descriptive
Body:     [What happens when they do it] — outcome, not feature description
CTA:      [Start the action] — specific verb, not "Next" or "OK"
```

**Examples:**

✓ Good:
```
Headline: Connect your first integration
Body:     Pull in customer data from your CRM, support tool, or analytics platform.
          Takes about 2 minutes.
CTA:      Connect Salesforce
```

✗ Bad:
```
Headline: Welcome to the Integrations Module
Body:     Our integrations functionality enables users to synchronize data
          bidirectionally across multiple third-party platforms and services.
CTA:      Get Started
```

What's wrong with the bad version: "Integrations Module" is internal jargon,
not what the user calls it. The body describes the feature, not the outcome.
"Get Started" tells the user nothing about what they're starting.

---

## Pattern 2: Tooltip

**Use:** Contextual help on hover or focus. Appears on every hover.

**Word budget:** 12 words maximum. One sentence. No CTA.

**Formula:** "[What this element does] in [context if needed]."

**Examples:**

✓ Good:
```
Health Score
"A weighted score based on usage, feature adoption, and support activity."
```

✓ Good:
```
Last 30 days
"Compared to the previous 30-day period."
```

✗ Bad:
```
Health Score
"The Health Score is a proprietary metric calculated by our advanced algorithm
that weighs multiple customer success indicators including but not limited to
product usage frequency, feature adoption rates, and support ticket volume."
```

That's a paragraph, not a tooltip. Move it to documentation.

---

## Pattern 3: Empty State

**Use:** When a section has no content yet. Permanent UI element.

**Word budget:**
- Headline: 6 words maximum
- Body: 30 words maximum
- CTA: 5 words maximum

**Formula:**
```
Headline: [Action verb] + [what to add/do]  — not "No [things] yet"
Body:     [What this section enables] + [why to act now]
CTA:      [Specific primary action]
```

**Examples:**

✓ Good — New pipeline:
```
Headline: Add your first deal
Body:     Track opportunities from first contact to close.
          See your pipeline, forecast, and close rate in one place.
CTA:      Add a deal
```

✓ Good — No results:
```
Headline: No results for "salesforce"
Body:     Try a different search term or check your spelling.
CTA:      Clear search
```

✗ Bad:
```
Headline: No deals found
Body:     You currently don't have any deals in your pipeline.
          Please add deals to begin tracking your sales opportunities.
CTA:      Get Started
```

"No deals found" is a system message. "Add your first deal" is an invitation.

---

## Pattern 4: Feature Announcement Banner

**Use:** Notifying existing users of a new feature. Shows once.

**Word budget:**
- Headline: 5 words maximum (after "New:")
- Body: 20 words maximum
- CTA: 4 words maximum

**Formula:**
```
"New: [Feature name]"
"[What the user can now do that they couldn't before] — one sentence, their outcome."
CTA: [Action that shows them the feature]
```

**Examples:**

✓ Good:
```
New: Bulk export
Download all your customer records as a single CSV in one click.
[Export now]
```

✗ Bad:
```
Exciting new feature! 🎉
We're thrilled to announce our brand new bulk export functionality which allows
you to export multiple records simultaneously.
[Learn more]
```

"Exciting" is for press releases. "We're thrilled" is filler. "Functionality"
is a feature word, not a user word. "Learn more" delays the action.

---

## Pattern 5: Checklist Item

**Use:** Step in an onboarding checklist. Each item is one action.

**Word budget:** 6 words maximum. Imperative verb + object. No body copy.

**Formula:** "[Verb] [your/the] [object]"

**Examples:**

✓ Good:
```
○ Connect your email account
○ Add your first contact
○ Create a pipeline stage
○ Invite a teammate
```

✗ Bad:
```
○ Email account connection and configuration
○ Initial contact creation step
○ Pipeline stage setup process
○ Team member invitation and onboarding
```

Checklists are to-do lists, not feature labels. Write them as tasks.

---

## Pattern 6: Upgrade Prompt

**Use:** When a user hits a plan limit or tries a locked feature.

**Word budget:**
- Headline: 8 words maximum
- Body: 20 words maximum
- CTA: 5 words maximum

**Formula:**
```
"[Feature name] is on the [Tier] plan"
"[One sentence: the specific value they'd unlock]"
CTA: "Upgrade to [Tier]"
```

**Examples:**

✓ Good:
```
Advanced analytics is on the Pro plan
Unlock cohort analysis, custom dashboards, and team-wide CSV export.
[Upgrade to Pro]
```

✗ Bad:
```
Upgrade Required
You need to upgrade your subscription plan to access this premium feature.
Please contact your account administrator to discuss upgrading options.
[Contact Us]
```

Never make a user feel blocked or punished. The tone should be "here's the
path forward" not "you can't do that."

---

## Anti-Patterns to Remove From Any Copy

| Word/Phrase | Why it fails | Replace with |
|-------------|-------------|-------------|
| "Simply" | Condescending — if it were simple they wouldn't need help | Just remove it |
| "Just" | Same as simply | Remove it |
| "Easy" | Patronising; also, nothing is easy to someone who's stuck | Describe what makes it fast |
| "Powerful" | Marketing word; meaningless in context | Describe the specific capability |
| "Get started" | Vague CTA | Name the specific first action |
| "Learn more" | Delays action | Link to the specific doc; or remove the link and put the info inline |
| "Click here" | No context | Name what they're clicking to |
| "We're excited to announce" | Filler | Skip straight to what changed |
| "Please" | Weakens instructions | Remove it; instructions don't need softening |
| Exclamation marks | Overly enthusiastic; erodes trust | Use sparingly; max 1 per flow |
