# Action Reference

All actions live in the `actions` array. Each action is a JSON object with an `action` field and action-specific fields.

## Target Strings — How They Resolve

All `target` / `origin` values resolve to a Puppeteer P-selector (verified on recordable 0.10.0 / Puppeteer 25.2.1):

| You write | recordable does | Example |
|---|---|---|
| `text:foo` | rewrites to `::-p-text(foo)` | `text:Create account` |
| `:text(foo)` anywhere | rewrites to `::-p-text(foo)` — composes with plain CSS | `button:text(Save)` |
| trailing `:nth(N)` | takes the Nth **visible** match (1-based, document order) | `text:Add comment:nth(2)` |
| `xpath=...` | routes to Puppeteer's built-in XPath handler | `xpath=//tbody/tr[3]/td[5]/button` |
| anything else | passes through verbatim as CSS — so native `:has()` works (Chrome 105+) | `tbody tr:has(span.text-amber-600) button` |

Two hard rules:

- **Never prefix a target with `css=`.** It is not stripped and Puppeteer has no `css` handler — the string reaches `querySelector` as invalid CSS and fails with "Could not find target".
- **Don't mix `:has()` with `:text()`** (`tbody tr:has(x) button:text(y)` fails — verified), and **don't combine `:text()` with `:nth()`** (`text:foo:nth(2)` is fine; `button:text(foo):nth(2)` is not). Pick one mechanism per target.

## Navigation

### `visit`

Full page navigation via Puppeteer's `page.goto(url)`.

```json
{ "action": "visit", "url": "http://localhost:5173/register" }
```

**Warning:** This triggers a full page reload. In SPAs, in-memory state is wiped. Use `click` on a nav link for post-login navigation instead. See SKILL.md "SPA Navigation" section.

### `click`

Click the first element matching the selector.

```json
{ "action": "click", "target": "text:Create account" }
{ "action": "click", "target": "#submit-button" }
{ "action": "click", "target": "[data-testid='export-btn']" }
```

Picks the **first match** in DOM order. If the text appears in a heading before the button, the heading wins. See SKILL.md "Selector Rules" section.

To take the second/third match instead, add a trailing `:nth(N)`:

```json
{ "action": "click", "target": "text:Export backup:nth(2)" }
```

For row-scoped targets, prefer structural `:has()` over positional `tr:nth-child(n)` when row order could change with the data:

```json
{ "action": "click", "target": "tbody tr:has(span.text-amber-600) button" }
```

### `hover`

Move the cursor to the element. Only visible when `cursor: true` is set in config.

```json
{ "action": "hover", "target": "text:Create account" }
{ "action": "hover", "target": "#username" }
```

Use before `click` to show the cursor moving to the target. Do NOT use `button:has-text(...)` — that's Playwright syntax. recordable's equivalent is `button:text(...)`; for "the button inside the row that has X", use structural `:has()`: `tbody tr:has(span.text-amber-600) button`.

## Input

### `type`

Type text into an input or textarea. Uses Puppeteer's `page.type()` which fires real `keydown`/`input`/`keyup` events.

```json
{ "action": "type", "target": "#username", "text": "demo@example.com" }
```

Typing speed is controlled by `config.typingSpeed` (chars per second, default 14).

### `clear`

Clear an input field.

```json
{ "action": "clear", "target": "#username" }
```

### `select`

Select an option in a `<select>` element.

```json
{ "action": "select", "target": "#country", "value": "FR" }
```

### `key`

Press a keyboard key.

```json
{ "action": "key", "target": "Enter" }
{ "action": "key", "target": "Escape" }
```

## Waiting

### `waitFor`

Wait for an element to reach a state. **Always use after navigation** to avoid racing the page render.

```json
{ "action": "waitFor", "target": "#username", "state": "visible", "timeout": 30000 }
```

States:
- `visible` — element is in the DOM and not hidden
- `hidden` — element is hidden or removed
- `attached` — element exists in the DOM (may be hidden)
- `detached` — element removed from the DOM

`timeout` is in milliseconds. Default is 30000 (30s).

**Sentinel pattern:** Use a selector that only appears on the destination page, not in the nav menu. E.g. after navigating to `/audit`, `waitFor text:Download JSON` (button only on /audit) instead of `text:Security log` (matches nav link + page heading).

### `wait`

Sleep for N milliseconds.

```json
{ "action": "wait", "ms": 1500 }
```

Use for:
- Giving async `useEffect` time to complete after a page mounts
- Holding a zoom for the viewer to read
- Pacing between actions

## Recording Control

### `pause` / `resume`

Pause and resume ffmpeg recording. The browser keeps running, but frames aren't captured.

```json
{ "action": "pause" },
{ "action": "click", "target": "text:Next" },
{ "action": "waitFor", "target": "text:Step 2", "state": "visible" },
{ "action": "resume" }
```

Use to hide transitions between pages/steps. **If ffmpeg is dead (ENOENT), `pause` hangs forever** — fix ffmpeg first.

### `start` / `end` / `split`

Segment markers for multi-segment recordings. Rarely needed — `pause`/`resume` is the common pattern.

## Zoom

### `zoom`

Zoom the viewport into an element.

```json
{ "action": "zoom", "level": 1.3, "origin": "#username" }
{ "action": "zoom", "level": 1.5, "origin": "text:Copied" }
```

`level` is a multiplier (1.0 = no zoom, 1.3 = 130%, 2.0 = 200%). `origin` is a selector that the zoom centers on.

Transition speed is controlled by `config.zoomDuration` (milliseconds, default 0 = instant).

### `resetZoom`

Reset zoom to 100%.

```json
{ "action": "resetZoom" }
```

Always pair `zoom` with a `resetZoom` later. Lingering zoom carries over to subsequent actions.

## Scrolling

### `scroll`

Scroll the page.

```json
{ "action": "scroll", "target": "bottom" }
{ "action": "scroll", "target": "top" }
{ "action": "scroll", "target": "#section-3" }
```

`top` and `bottom` scroll the main viewport. A selector scrolls until that element is in view.

## Clips

### `insert`

Splice a pre-recorded MP4 clip into the output video.

```json
{ "action": "insert", "path": "clips/desktop.mp4", "fadeIn": 500, "fadeOut": 500 }
```

`path` is relative to the script file's directory. `fadeIn`/`fadeOut` are in milliseconds.

**Fails if the file doesn't exist** — remove `insert` actions if you don't have the clip files yet.

## Configuration

### `setConfig`

Change config mid-script.

```json
{ "action": "setConfig", "config": { "typingSpeed": 8 } }
```

Rarely needed — set config in the top-level `config` block instead.

## Audio

### `audio`

Add an audio track.

```json
{ "action": "audio", "path": "narration.mp3", "fadeIn": 500, "fadeOut": 500 }
```

Rarely used for demo videos.

## Choosing Between Similar Actions

| You want to... | Use |
|---|---|
| Navigate to a new URL (pre-login) | `visit` |
| Navigate within an SPA (post-login) | `click` on a nav link |
| Wait for a page to render | `waitFor` with a page-unique sentinel |
| Show the cursor moving to an element | `hover` (with `cursor: true`) |
| Highlight a success toast | `zoom` 1.3x + `wait` 1000ms + `resetZoom` |
| Hide a navigation transition | `pause` before nav, `resume` after `waitFor` |
| Type into a form field | `type` (with `#id` selector) |
| Click a button whose text matches a heading | `click [data-testid='...']` |
| Target the row whose state matches (amber stock, flagged row, …) | `tbody tr:has(span.text-amber-600) button` |
