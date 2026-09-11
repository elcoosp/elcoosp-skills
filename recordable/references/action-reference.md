# Action Reference

All actions live in the `actions` array. Each action is a JSON object with an `action` field and action-specific fields.

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

### `hover`

Move the cursor to the element. Only visible when `cursor: true` is set in config.

```json
{ "action": "hover", "target": "text:Create account" }
{ "action": "hover", "target": "#username" }
```

Use before `click` to show the cursor moving to the target. Do NOT use `button:has-text(...)` — Puppeteer doesn't support it.

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
