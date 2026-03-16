# WCAG 2.1 AA Accessibility Checklist

DX verifies every design against this checklist before handing off to EL.
WCAG 2.1 AA is the legal standard in most jurisdictions and the minimum
acceptable level for any professional product.

Target: WCAG 2.1 Level AA. Level AAA is aspirational for specific features
(e.g., captions for live video); do not block release for AAA-only criteria.

---

## 1. Colour and Contrast

**1.1 Text contrast — Success Criterion 1.4.3 (AA)**

| Text type | Minimum contrast ratio |
|-----------|----------------------|
| Normal text (< 18pt / < 14pt bold) | 4.5:1 |
| Large text (≥ 18pt / ≥ 14pt bold) | 3:1 |
| Disabled text | No requirement (exempt) |

Tool: Use the browser's DevTools accessibility panel, or [Colour Contrast Analyser](https://www.tpgi.com/color-contrast-checker/).

**1.2 UI component contrast — Success Criterion 1.4.11 (AA)**

| Element | Minimum contrast ratio |
|---------|----------------------|
| Input borders | 3:1 against background |
| Focus indicators | 3:1 against adjacent colours |
| Button borders | 3:1 if border alone distinguishes the button |
| Icons conveying meaning | 3:1 |

**1.3 Colour alone is not the only indicator — Success Criterion 1.4.1 (A)**

Never use colour as the only way to convey information.

- Error states: red border + error message text + error icon ✓
- Error states: red border only ✗
- Required fields: asterisk (*) + label + aria-required ✓
- Required fields: red label only ✗
- Charts: colour + pattern/label ✓; colour only ✗

---

## 2. Keyboard Navigation

**2.1 All functionality available by keyboard — Success Criterion 2.1.1 (A)**

Test without a mouse:
- [ ] Tab moves focus through all interactive elements in logical order
- [ ] Shift+Tab moves focus backward
- [ ] Enter/Space activates buttons and checkboxes
- [ ] Arrow keys navigate within components (menus, radio groups, sliders, tabs)
- [ ] Escape closes modals, dropdowns, and tooltips
- [ ] Focus never gets trapped in a component the user cannot leave (unless intentional — e.g., open modal)

**2.2 Visible focus indicator — Success Criterion 2.4.7 (AA) / 2.4.11 (AA, WCAG 2.2)**

- [ ] Every focusable element has a visible focus ring
- [ ] Focus ring has ≥ 3:1 contrast against adjacent colours
- [ ] Focus ring is not `outline: none` with no replacement

---

## 3. Screen Reader Support

**3.1 Images have alt text — Success Criterion 1.1.1 (A)**

- [ ] Informative images: `alt="descriptive text explaining what the image shows"`
- [ ] Decorative images: `alt=""` (empty alt, not missing)
- [ ] Icons that convey meaning: `aria-label` on the icon's parent or `title` attribute
- [ ] Complex images (charts, diagrams): long description provided in nearby text or `aria-describedby`

**3.2 Form labels — Success Criterion 1.3.1 (A)**

- [ ] Every input has a `<label>` associated via `for`/`id`
- [ ] Error messages associated via `aria-describedby`
- [ ] Required fields have `aria-required="true"`
- [ ] Placeholder text is not the only label (placeholder disappears on input)

**3.3 Page structure — Success Criterion 1.3.1 (A)**

- [ ] Page has one `<h1>` (the page title)
- [ ] Heading hierarchy is logical: H1 → H2 → H3 (no skipping)
- [ ] Landmark regions: `<main>`, `<nav>`, `<header>`, `<footer>` used correctly
- [ ] Lists use `<ul>` / `<ol>` / `<dl>` (not just styled `<div>` elements)

**3.4 Dynamic content announcements**

- [ ] Loading states: `aria-live="polite"` on the region that updates
- [ ] Error messages: `role="alert"` for immediate announcements
- [ ] Status messages (saved, deleted): `aria-live="polite"` region
- [ ] Modals: focus moves to modal on open; returns to trigger on close; `aria-modal="true"`
- [ ] Expandable sections: `aria-expanded="true/false"` on the trigger

**3.5 Link and button names — Success Criterion 4.1.2 (A)**

- [ ] Every button has a text label or `aria-label`
- [ ] "Click here" and "Read more" links have `aria-label` with context ("Read more about [topic]")
- [ ] Icon-only buttons have `aria-label`

---

## 4. Motion and Animation

**4.1 Respect prefers-reduced-motion — Success Criterion 2.3.3 (AAA, but best practice at AA)**

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

- [ ] All animations respect this media query
- [ ] Parallax scrolling is disabled when reduced motion is preferred
- [ ] Auto-playing content (carousels, videos) has a pause control

---

## 5. Responsive and Zoom

**5.1 No horizontal scroll at 320px viewport — Success Criterion 1.4.10 (AA)**

- [ ] Content reflows at 320px width without horizontal scrolling
- [ ] Touch targets are ≥ 44×44px — Success Criterion 2.5.5 (AA)

**5.2 Text resize — Success Criterion 1.4.4 (AA)**

- [ ] Text can be resized to 200% without loss of content or functionality
- [ ] Do not use `px` for font sizes on body text — use `rem` so browser zoom works
- [ ] Containers do not overflow or clip at 200% zoom

---

## 6. Quick Test Protocol

Before every design handoff, run these five tests:

```
1. Tab through the entire screen without using a mouse.
   Can you reach and activate every interactive element?

2. View the screen with a screen reader (VoiceOver on Mac, NVDA on Windows).
   Does the read-out order match the visual order?
   Are all interactive elements announced correctly?

3. Check all text contrast ratios.
   Is every text/background combination ≥ 4.5:1?

4. Disable all CSS.
   Does the content still make sense in its reading order?

5. Enable browser zoom to 200%.
   Does the layout still work without horizontal scrolling?
```

Failing any of these is a blocker for handoff, equivalent to a P1 bug.
