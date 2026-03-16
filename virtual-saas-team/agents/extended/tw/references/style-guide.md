# Style Guide — Voice, Tone, Formatting, and Terminology

TW applies this guide to all documentation. Consistency across docs builds
trust. When the writing style is consistent, users focus on the content
rather than noticing the writing.

---

## Voice and Tone

### Voice (consistent — doesn't change)

**Direct.** State things clearly. Do not hedge or equivocate unless genuine
uncertainty exists.
- ✓ "Click Save to apply your changes."
- ✗ "You may want to consider clicking Save, which could potentially apply your changes."

**Specific.** Name the thing. Describe the action. Avoid vague references.
- ✓ "In the Billing settings, click Update payment method."
- ✗ "In the settings area, find the billing options."

**Second person.** Address the reader as "you." Not "the user" or "one."
- ✓ "You can export your data as a CSV."
- ✗ "Users can export their data as a CSV."
- ✗ "One can export data as a CSV."

**Active voice.** Subject does the action.
- ✓ "Click the button to confirm."
- ✗ "The button should be clicked to confirm."

### Tone (context-dependent)

| Context | Tone | Example |
|---------|------|---------|
| Getting started / quickstarts | Encouraging, energetic | "You're 5 minutes from your first integration." |
| Reference docs (API, config) | Precise, neutral | "The `limit` parameter accepts integers from 1 to 100." |
| Troubleshooting | Empathetic, solution-focused | "If you see this error, here's what to check first." |
| Warning / caution | Clear, serious (not alarming) | "This action cannot be undone." |
| Error messages | Calm, specific, actionable | "We couldn't connect to your CRM. Check your API key in Settings → Integrations." |

---

## Formatting Rules

### Headings

Use sentence case. Not Title Case.
- ✓ "How to connect your CRM"
- ✗ "How To Connect Your CRM"

Use headings to answer questions or name tasks, not to label sections.
- ✓ "What happens when you delete an account"
- ✗ "Account deletion overview"

Heading hierarchy:
```
H1 — Page title (one per page)
H2 — Major section
H3 — Subsection within H2
H4 — Rarely needed; use sparingly
Never skip a level (H1 → H3 with no H2)
```

### Lists

Use bullet lists for items with no inherent order. Use numbered lists when
sequence matters (instructions, steps) or when you'll reference items by number.

**Bullet list rules:**
- All items parallel in grammar (all nouns, or all verb phrases — not mixed)
- No more than 7 items. More than 7 = consider restructuring.
- Each item one sentence max (use prose for longer items)
- No full stops on bullets unless items are full sentences — pick one and be consistent

**Numbered list rules:**
- Use for procedures: one action per step
- Each step starts with a verb
- Include the expected result for each step where helpful

### Code Formatting

Inline code: use backticks for `variable_names`, `function_names`, `file_names`,
`endpoint_paths`, and any string the user must type exactly.

Code blocks: use fenced code blocks with language specified:
```python
# Example Python code
response = client.get("/users", params={"limit": 10})
```

**Do not use code formatting for:**
- UI element names ("Click the **Save** button" — bold is correct, not `Save`)
- Product feature names (Analytics, not `Analytics`)

### UI Element References

| Element | Format | Example |
|---------|--------|---------|
| Button labels | Bold | Click **Save** |
| Menu items | Bold, > for paths | Go to **Settings > Billing** |
| Field names | Bold | Enter your email in the **Email address** field |
| Page names | Regular text, capitalised | Open the Integrations page |
| Keyboard shortcuts | `code` | Press `Cmd+S` to save |

### Callouts

Use callouts sparingly. If everything is called out, nothing is.

```
> **Note:** For additional context that's helpful but not critical.

> **Important:** For information that will affect the user if missed.

> **Warning:** For actions that cannot be undone or have significant consequences.

> **Tip:** For a faster or better way to do something.
```

---

## Terminology — Approved Terms

Always use the product's own terminology consistently. When terms conflict
with common usage, the product term wins in product documentation.

### Use vs Avoid

| Use | Avoid | Reason |
|-----|-------|--------|
| Click | Press, hit, select (for mouse actions) | "Click" is the precise term for mouse interaction |
| Tap | Click, press (for mobile/touch) | "Tap" is the precise term for touch |
| Select | Check, tick (for checkboxes) | "Select" works across all input types |
| Enter | Type, input, put in | "Enter" is more precise |
| Sign in | Log in, login | Consistent with most modern product conventions |
| Sign out | Log out, logout | Consistent |
| Cancel | Abort, stop, exit | "Cancel" is the standard UI term |
| Remove | Delete (when data isn't destroyed) | "Delete" implies permanent destruction |
| Delete | Remove (when data is permanently destroyed) | Reserve "delete" for permanent actions |
| [Product name] | "the platform," "the tool," "the system" | Always use the actual product name |

### Numbers

Spell out numbers one through nine. Use digits for 10 and above.
Exception: always use digits before units (5 minutes, 3 MB, 1 hour).

### Dates and Times

ISO 8601 in technical contexts: `2026-04-15`
Human-readable in documentation: April 15, 2026 (not 15 April or 4/15/26)
Timezone: always specify (UTC, not "server time")

---

## Things That Are Never Acceptable

- Referring to users as "the user" instead of "you"
- Jargon that hasn't been defined (use a glossary for technical terms on first use)
- Passive voice in instructions ("The button should be clicked")
- Ambiguous pronouns ("When it is configured, it should work")
- Future tense for instructions ("You will then click") — use present tense
- Making the user feel stupid ("Simply click..." / "Just press..." — nothing is simple to someone who's stuck)
