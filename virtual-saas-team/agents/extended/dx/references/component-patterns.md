# Component Patterns and Anti-Patterns

DX uses this reference when specifying components in DX-001 (Design System)
and DX-002 (Feature Design Spec). Each pattern includes: when to use it,
the states to specify, and the common anti-patterns to avoid.

All values reference design tokens from DX-001. Never hardcode colours,
spacing, or typography outside the token system.

---

## Form Components

### Text Input

**States to always specify:** default, focus, filled, error, disabled, read-only.

**Pattern:**
```
Default:   1px border-tertiary; bg-primary; placeholder in text-tertiary
Focus:     2px border-primary (or brand colour); shadow-ring
Filled:    Same as default; text in text-primary
Error:     1px border-danger; error message below in text-danger; icon prefix optional
Disabled:  bg-secondary; text-secondary; cursor: not-allowed; no interaction
Read-only: No border; text-primary; cursor: default
```

**Anti-patterns:**
- ❌ Error state shown only by red border — add an error message. Colour alone is not accessible.
- ❌ Placeholder text as the label — the placeholder disappears when the user types. Always use a visible label above or floating label pattern.
- ❌ No success state for validated fields — when validation passes, show it (green checkmark or success text). Do not leave the user guessing.

**Accessibility:**
- `<label>` must be associated with input via `for`/`id` or aria-labelledby
- Error message must be associated via `aria-describedby`
- Required fields marked with `aria-required="true"` and a visible indicator (not just "*")

---

### Dropdown / Select

**States:** default, open, selected, search (if filterable), empty-state, error, disabled.

**Pattern:**
```
Default:   Looks like text input; chevron-down icon; selected value in text-primary
Open:      Elevated card (shadow-md); options list; max-height with scroll
Selected:  Checkmark on active option; value in trigger reflects selection
Search:    Text input at top of dropdown list; filters as you type
Empty:     "No results for [search term]" — never an empty white box
Error:     Same as text input error pattern
```

**Anti-patterns:**
- ❌ Native `<select>` for complex dropdowns — style it custom for consistency
- ❌ Dropdown that closes on scroll outside — should close only on Escape, click outside, or selection
- ❌ More than 10 options without search — add a search field
- ❌ No keyboard navigation — Tab to focus, Space/Enter to open, arrows to navigate, Enter to select, Escape to close

---

### Checkbox and Radio

**States:** unchecked, checked, indeterminate (checkbox only), disabled, error.

**Anti-patterns:**
- ❌ Click target limited to the visual checkbox — the entire label row should be clickable
- ❌ Checkbox group with no "select all" option when there are > 5 items
- ❌ Radio group where "none" is a valid option but there's no way to deselect — add an explicit "None" option

---

## Feedback Components

### Toast / Notification

**Use for:** Transient feedback on user actions (saved, deleted, error occurred).
**Do not use for:** Errors requiring user action (use inline error) or information critical to completing a task (use inline alert).

**Pattern:**
```
Position:   Bottom-right (desktop); bottom (mobile)
Duration:   Success/info: 4 seconds; Error: persistent until dismissed
Types:      success (green), error (red), warning (amber), info (blue)
Anatomy:    Icon + title + optional description + optional action link + dismiss
Max stack:  3 toasts visible at once; queue additional
```

**Anti-patterns:**
- ❌ Toasts for errors the user must act on — they disappear before the user reads them
- ❌ Toast with no dismiss — always provide a way to manually dismiss
- ❌ Toast stacking indefinitely — cap at 3; collapse or scroll older ones

---

### Inline Alert

**Use for:** Persistent messages that affect the current page state — form errors,
system status messages, warnings about irreversible actions.

**Pattern:**
```
Types:      info, success, warning, error
Anatomy:    Coloured left border or bg-tint + icon + message + optional CTA
Position:   Above the affected content (form errors above the form; page alert at top)
Persistence: Stays until user dismisses or condition resolves
```

**Anti-patterns:**
- ❌ Using a toast where an inline alert is needed — if the user needs to act on the message, it must stay visible
- ❌ Alert with no icon — colour + icon together ensure accessibility for colourblind users

---

### Empty State

**Anatomy:** Icon/illustration + headline (action-oriented) + body (what can be done here) + primary CTA.

**Pattern:**
```
New section (no data yet):
  Icon:     Relevant to the section (not a generic placeholder)
  Headline: "Add your first [thing]" — tell them what to do
  Body:     1-2 sentences: what this section enables; why they should care
  CTA:      Primary action button — specific label, not "Get started"

No search results:
  Icon:     Search/magnifier
  Headline: "No results for '[search term]'"
  Body:     Suggest: check spelling, try different terms, or clear filters
  CTA:      "Clear filters" or "Search all [items]"

Error state:
  Icon:     Warning
  Headline: "Couldn't load [section name]"
  Body:     Brief explanation; not a technical error message
  CTA:      "Try again" button
```

**Anti-patterns:**
- ❌ Blank white space — always show an empty state component
- ❌ Generic "No data" text with no guidance on what to do
- ❌ Empty state that doesn't match the section context

---

## Navigation Components

### Primary Navigation

**Anti-patterns:**
- ❌ More than 7 top-level items — reorganise into groups if needed
- ❌ Active state indicated only by colour — add bold weight or underline for accessibility
- ❌ Navigation that re-orders on mobile — consistent position reduces cognitive load

### Breadcrumbs

**Use for:** Any page more than 2 levels deep.

**Pattern:**
```
Home / Section / Sub-section / Current page
Last item is not a link (it's the current page)
Truncate long paths: Home / … / Sub-section / Current
```

---

## Data Display Components

### Table

**States:** loading, empty, error, populated (with sort, filter, pagination).

**Pattern:**
```
Header:       Bold labels; sortable columns show sort icon (both directions available)
Rows:         Hover state (bg-secondary); click-to-select or click-to-navigate
Actions:      Row actions appear on hover or in a kebab menu; destructive actions last
Pagination:   Show "1–25 of 347 results"; next/prev; page size selector
Responsive:   On mobile, either horizontal scroll or collapse to card view
```

**Anti-patterns:**
- ❌ Table with no empty state — show a message, not empty rows
- ❌ Sortable columns with no visual indication of current sort
- ❌ More than 8 visible columns — hide secondary columns behind a column picker
- ❌ Destructive actions (delete, remove) without confirmation — especially in tables where mis-clicks are common

### Data Card / Metric

**Use for:** Summary numbers on dashboards.

**Pattern:**
```
Label:    12–13px; text-secondary; above the number
Value:    24–32px; weight-500; text-primary
Delta:    14px; colour-coded (green = positive, red = negative); arrow indicator
Trend:    Optional sparkline; 4–6 data points
```

**Anti-patterns:**
- ❌ More than 4–6 metric cards in a row — group or prioritise
- ❌ No context for the metric — "$47,231" alone means nothing; add "MRR" and "▲12% vs last month"
