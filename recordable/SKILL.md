---
name: recordable-demo
description: |
  Create and debug `recordable` JSON scripts for recording browser-based demo videos of web applications. `recordable` is a CLI tool (built on Puppeteer + ffmpeg) that drives a real Chrome instance through a declarative JSON script — visiting pages, typing, clicking, zooming, and producing an MP4. Use this skill whenever the user wants to: record a web app demo or walkthrough video, write a `demos/*.json` recordable script, debug a failing `pnpm rec:demo` or `recordable demos/foo.json` run, automate browser interactions for a screencast, or fix recordable errors like "Could not find target", "ffmpeg ENOENT", or "TimeoutError: Waiting for selector". Also triggers when the user mentions: recordable, puppeteer recording, demo video, screen recording automation, browser action scripting, `rec:demo`, or `pnpx recordable`.
---

# recordable — Browser Demo Video Recorder

`recordable` drives a real Chrome browser through a declarative JSON script and produces an MP4 video. It's built on **Puppeteer** (not Playwright) + **ffmpeg-static**. This skill tells you how to write scripts that actually work, and how to debug the common failures.

## Prerequisites

Before any `recordable demos/foo.json` run, the environment must have:

1. **A running dev server** for the web app (e.g. `pnpm --filter @vautr/web dev` on `:5173`)
2. **A running backend API server** if the app needs one (e.g. `vautr-server` on `:8080`)
3. **Chrome installed for Puppeteer** — if missing, run `pnpx puppeteer browsers install chrome`
4. **ffmpeg binary resolvable** — see [troubleshooting.md](./references/troubleshooting.md) if you see `ffmpeg ENOENT`
5. **Fresh browser state** — clear site data (DevTools → Application → Storage → Clear site data) so no stale session tokens interfere
6. **Fresh DB** if the app uses a database — wipe it so the script's register/login flow doesn't hit duplicate-key errors

## JSON Script Structure

Every recordable script is a JSON file with this shape:

```json
{
  "$schema": "https://raw.githubusercontent.com/paragramagency/recordable/main/recordable.schema.json",
  "config": {
    "viewport": { "width": 1920, "height": 1080 },
    "fps": 30,
    "headless": false,
    "typingSpeed": 14,
    "actionDelay": 400,
    "cursor": true,
    "zoomDuration": 400,
    "outputDir": "output",
    "outputName": "my-demo"
  },
  "actions": [
    { "action": "pause" },
    { "action": "visit", "url": "http://localhost:5173/register" },
    { "action": "waitFor", "target": "#username", "state": "visible" },
    { "action": "resume" },
    { "action": "type", "target": "#username", "text": "demo@example.com" },
    { "action": "click", "target": "text:Create account" }
  ]
  }
}
```

### Config keys that matter

| Key | Default | Why you'd change it |
|---|---|---|
| `cursor` | `false` | Set `true` to show a visible cursor (makes `hover` actions visible in the video) |
| `zoomDuration` | `0` (instant) | Set to `300`–`500` for smooth animated zoom transitions instead of jump cuts |
| `actionDelay` | `500` | Lower to `300`–`400` for a snappier demo; raise to `800` if actions race the page |
| `typingSpeed` | `14` | Chars per second; lower (e.g. `8`) for dramatic effect, higher (e.g. `20`) for speed |
| `headless` | `false` | Keep `false` so you can watch the browser during recording |
| `outputDir` | `"output"` | Where the MP4 is written (relative to the script file's directory) |
| `outputName` | `"demo"` | Base filename; recordable appends a timestamp |

## Actions

All actions live in the `actions` array. Here are the ones you'll use most:

| Action | Required fields | What it does |
|---|---|---|
| `visit` | `url` | Full page navigation (`page.goto`). **Triggers a full page reload** — see SPA warning below |
| `click` | `target` | Click the first element matching the selector |
| `hover` | `target` | Move the cursor to the element (visible when `cursor: true`) |
| `type` | `target`, `text` | Type text into an input/textarea |
| `waitFor` | `target`, `state`, `timeout` | Wait for an element to be `visible`/`hidden`/`attached`/`detached`. **Always use this after navigation** to avoid racing the page render |
| `wait` | `ms` | Sleep for N milliseconds |
| `zoom` | `level`, `origin` | Zoom the viewport into an element (e.g. `1.3` = 130%). `origin` is a selector |
| `resetZoom` | — | Reset zoom to 100% |
| `pause` | — | Pause ffmpeg recording (the browser keeps running, but frames aren't captured). **Hangs if ffmpeg is dead** — see troubleshooting |
| `resume` | — | Resume ffmpeg recording |
| `scroll` | `target` (`"top"` / `"bottom"` / selector) | Scroll the page |
| `insert` | `path`, `fadeIn`, `fadeOut` | Splice a pre-recorded MP4 clip into the output. **Fails if the file doesn't exist** |

For the full action reference + edge cases, see [references/action-reference.md](./references/action-reference.md).

## Selector Rules — Read This Or You Will Fail

`recordable` uses **Puppeteer** under the hood. Puppeteer's selector engine is **not the same as Playwright's**. This is the #1 source of failures.

### What works

| Pattern | Example | Notes |
|---|---|---|
| `text:foo` | `text:Create vault` | Substring text match. **Picks the first match in DOM order** — see warning below |
| `#id` | `#username` | Standard CSS ID selector |
| `.class` | `.btn-primary` | Standard CSS class selector |
| `[attr='value']` | `[data-testid='export-btn']` | Standard CSS attribute selector — **use this when text is ambiguous** |
| `css=...` | `css=button.primary` | Explicit CSS engine (rarely needed) |

### What does NOT work

| Pattern | Why it fails | Fix |
|---|---|---|
| `button:has-text("foo")` | `:has-text()` is **Playwright-specific**. Puppeteer doesn't understand it → "Could not find target" | Use `[data-testid='...']` on the button, or `text:foo` if unambiguous |
| `text="foo"` (exact) | Puppeteer's `text=` is "contains", not "exact" — matches multiple elements | Same as above |
| `xpath=//button[...]` | Puppeteer's `click()` doesn't support XPath directly | Use CSS or `text:` instead |

### The "text matches both heading and button" trap

When the same text appears in both a **heading** (`<h1>`, `<h2>`, `<h3>`) and a **button** (`<button>`), `text:foo` matches the heading first (headings come before buttons in DOM order). Your click lands on the non-clickable heading and does nothing.

**Example:** A card has `<h3>Export backup</h3>` and `<button>Export backup</button>`. `click text:Export backup` clicks the heading → nothing happens.

**Fixes (in order of preference):**
1. Add `data-testid="export-backup-button"` to the `<button>` in the app code, then `click [data-testid='export-backup-button']`
2. Use a CSS structural selector like `div.grid > div:first-child button`
3. Use a different text that only appears on the button (e.g. the button's `aria-label`)

### The "text matches both nav link and page heading" trap

In SPA layouts with a sidebar, the nav link text often matches the page heading text (e.g. "Dashboard" appears in both the sidebar link and the `<h1>`). For `waitFor`, this is fine — either match satisfies the visibility check. For `click`, you usually want the nav link (first match), which works. But for `waitFor` after clicking the nav link, the selector matches **immediately** (the nav link is already visible) — so it doesn't actually wait for the new page to render.

**Fix:** Use a sentinel that only appears on the destination page, not in the nav. E.g. after navigating to `/dashboard`, `waitFor text:Recent projects` (the card title) instead of `text:Dashboard` (the nav link + heading).

## SPA Navigation — `visit` vs `click`

This is critical for single-page apps (React/Vue/Svelte with client-side routing).

### `visit` = full page reload

`{ "action": "visit", "url": "http://localhost:5173/dashboard" }` tells Puppeteer to `page.goto(url)`. This **reloads the entire JS bundle**. In SPAs, this means:
- The app's in-memory state is wiped
- `vaultStore` / `authStore` / similar reset to their initial values
- If the app doesn't restore session from `IndexedDB` / `localStorage` on mount, the user appears logged out
- The route guard redirects to `/login`
- Your subsequent `waitFor text:Dashboard` times out

**When `visit` is OK:** The initial page load (e.g. `visit /register` before login). No session to lose.

**When `visit` breaks:** Any post-login navigation in an SPA that doesn't restore session on reload.

### `click` on a nav link = client-side navigation

`{ "action": "click", "target": "text:Dashboard" }` clicks the sidebar link. In SPAs, the router handles this **without reloading the page**. In-memory state survives. This is what you want for all post-login navigation.

### Pattern: post-login navigation in an SPA

```json
{ "action": "pause" },
{ "action": "click", "target": "text:Projects" },
{ "action": "waitFor", "target": "text:New project", "state": "visible", "timeout": 30000 },
{ "action": "resume" }
```

The `pause`/`resume` bracket hides the brief transition between routes. The `waitFor` uses a page-unique sentinel (`New project` button only exists on `/projects`).

## Recording Patterns

### Pattern: form fill + submit

```json
{ "action": "zoom", "level": 1.3, "origin": "#username" },
{ "action": "type", "target": "#username", "text": "demo@example.com" },
{ "action": "type", "target": "#password", "text": "correct horse battery staple" },
{ "action": "resetZoom" },
{ "action": "hover", "target": "text:Create account" },
{ "action": "pause" },
{ "action": "click", "target": "text:Create account" },
{ "action": "waitFor", "target": "text:Dashboard", "state": "visible", "timeout": 60000 },
{ "action": "resume" }
```

The `pause` before the click + `resume` after the destination appears hides the navigation flash. Zoom into the form once, type all fields, reset. Hover before click for visible cursor movement.

### Pattern: click a button that appears after a form submits

After `click text:Save item`, the form unmounts and a detail view mounts. The detail view has a `Copy` button that the form didn't. If you `click text:Copy` immediately, the form is still mounted and `Copy` doesn't exist yet — or worse, it matches a different element.

**Fix:** `waitFor` a sentinel that only exists on the detail view before clicking:

```json
{ "action": "click", "target": "text:Save item" },
{ "action": "waitFor", "target": "text:Share", "state": "visible", "timeout": 15000 },
{ "action": "wait", "ms": 2000 },
{ "action": "click", "target": "text:Copy" }
```

The 2000ms wait gives any async `useEffect` (e.g. `reveal(uuid)` fetching from the server) time to complete before the click.

### Pattern: zoom on key results

When a toast or success message appears, zoom into it to highlight:

```json
{ "action": "waitFor", "target": "text:Copied", "state": "visible", "timeout": 5000 },
{ "action": "zoom", "level": 1.3, "origin": "text:Copied" },
{ "action": "wait", "ms": 1000 },
{ "action": "resetZoom" }
```

## Zoom Philosophy — Don't Overdo It

Too much zoom in/out causes nausea. Rules of thumb:

- **One zoom per page arrival** (1.2x, ~800-1000ms, then reset) — shows "we're on a new page"
- **No zoom on button clicks** — hover + click is enough; the cursor shows where the action is
- **No zoom on form fields** — type is enough; zoom in/out around typing is jarring
- **Zoom on key results** (1.3x, ~1000ms) — highlights success toasts
- **Never double-zoom** — no "zoom 1.2x → reset → zoom 1.4x" on the same element

## Common Errors — Quick Fix Table

| Error | Cause | Fix |
|---|---|---|
| `ffmpeg: Error: spawn .../ffmpeg-static/ffmpeg ENOENT` | `ffmpeg-static` postinstall didn't run (pnpm blocked it) | `pnpm config set approve-builds ffmpeg-static` then reinstall, OR symlink `ln -sf /opt/homebrew/bin/ffmpeg .../ffmpeg-static/ffmpeg` |
| `Could not find target: "button:has-text(...)"` | `:has-text()` is Playwright-only, recordable uses Puppeteer | Use `[data-testid='...']` or `text:foo` |
| `TimeoutError: Waiting for selector text:foo` | The element doesn't exist on the page (yet, or at all) | Check: is the page loaded? Is the text spelled exactly? Does it appear in a heading that the script already passed? |
| `Could not launch Chromium` | Puppeteer's Chrome isn't installed | `pnpx puppeteer browsers install chrome` |
| Script hangs after `Zoom reset` | The next action is `pause` and ffmpeg is dead (ENOENT earlier) | Fix ffmpeg first (see row 1) |
| `insert: file not found: .../clips/foo.mp4` | An `insert` action references a clip that doesn't exist | Remove the `insert` actions, or create the clip files |
| After `visit /dashboard`, bounced to `/login` | SPA doesn't restore session on reload | Replace `visit` with `click text:Dashboard` (client-side nav) |
| `text:foo matched 2 elements; using the first` | The text appears in multiple elements (heading + button, nav + heading) | Check if the first match is what you want to click. If not, use a unique selector |

For deeper debugging, see [references/troubleshooting.md](./references/troubleshooting.md).

## Full Example Script

Here's a minimal working script for a register → dashboard flow:

```json
{
  "$schema": "https://raw.githubusercontent.com/paragramagency/recordable/main/recordable.schema.json",
  "config": {
    "viewport": { "width": 1920, "height": 1080 },
    "fps": 30,
    "headless": false,
    "typingSpeed": 14,
    "actionDelay": 400,
    "cursor": true,
    "zoomDuration": 400,
    "outputDir": "output",
    "outputName": "demo"
  },
  "actions": [
    { "action": "pause" },
    { "action": "visit", "url": "http://localhost:5173/register" },
    { "action": "waitFor", "target": "#username", "state": "visible" },
    { "action": "resume" },
    { "action": "zoom", "level": 1.3, "origin": "#username" },
    { "action": "type", "target": "#username", "text": "demo@example.com" },
    { "action": "type", "target": "#password", "text": "correct horse battery staple" },
    { "action": "resetZoom" },
    { "action": "hover", "target": "text:Create account" },
    { "action": "pause" },
    { "action": "click", "target": "text:Create account" },
    { "action": "waitFor", "target": "text:Dashboard", "state": "visible", "timeout": 60000 },
    { "action": "resume" },
    { "action": "wait", "ms": 2000 }
  ]
}
```

## When The User Asks You To "Write A Demo Script"

1. Ask what app they're recording and what flow they want to show
2. Check if the app is an SPA (React/Vue/Svelte) — if so, use `click` for post-login navigation, not `visit`
3. Write the JSON with: `pause`/`resume` around navigation, `waitFor` with page-unique sentinels after each navigation, `hover` before clicks, `zoom` only on page arrival + key results
4. For any button whose text also appears in a heading, tell the user to add `data-testid` to the button in the app code, and use `[data-testid='...']` in the script
5. Remind them to: start the dev server + backend, clear browser site data, wipe the DB if the flow includes register/login
6. If a run fails, read the error from the table above and apply the fix

## Reference Files

- [references/action-reference.md](./references/action-reference.md) — Full action reference with all fields and edge cases
- [references/troubleshooting.md](./references/troubleshooting.md) — Deep debugging guide for ffmpeg, Chromium, session, and selector issues
