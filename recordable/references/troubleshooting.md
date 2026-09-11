# Troubleshooting

Deep debugging guide for the most common recordable failures.

## ffmpeg ENOENT

### Symptom

```
[Recordable] ffmpeg: Error: spawn /path/to/node_modules/.pnpm/ffmpeg-static@5.3.0/node_modules/ffmpeg-static/ffmpeg ENOENT
[Recordable] Record  start
... script runs but hangs at the next `pause` action ...
```

### Cause

`recordable` uses the `ffmpeg-static` npm package, which has a postinstall script that downloads a prebuilt ffmpeg binary into the package folder. The postinstall is failing silently — usually because:

1. **pnpm blocked the postinstall** (pnpm's `onlyBuiltDependencies` / `approve-builds` feature). This is the most common cause.
2. Network failure (GitHub rate limit, corporate proxy, stale CA bundle)
3. The `ffmpeg-static` version doesn't have a prebuilt binary for your platform

The `ENOENT` means the binary file doesn't exist on disk. Your `brew install ffmpeg` at `/opt/homebrew/bin/ffmpeg` is irrelevant — `recordable` imports `ffmpeg-static` directly and spawns that specific path.

### Fix — Option 1: Approve the build (pnpm)

```bash
pnpm config set approve-builds ffmpeg-static
pnpm install
```

Or edit `package.json`:

```json
{
  "pnpm": {
    "onlyBuiltDependencies": ["ffmpeg-static"]
  }
}
```

Then `pnpm install`. Verify:

```bash
ls -l node_modules/.pnpm/ffmpeg-static@*/node_modules/ffmpeg-static/ffmpeg
# Should be ~70MB, not "no such file"
```

### Fix — Option 2: Re-run postinstall

```bash
pnpm rebuild ffmpeg-static
```

### Fix — Option 3: Symlink system ffmpeg

If the postinstall can't fetch (network/proxy), symlink brew's ffmpeg:

```bash
ln -sf /opt/homebrew/bin/ffmpeg \
  node_modules/.pnpm/ffmpeg-static@5.3.0/node_modules/ffmpeg-static/ffmpeg
```

Verify:

```bash
node_modules/.pnpm/ffmpeg-static@5.3.0/node_modules/ffmpeg-static/ffmpeg -version | head -1
# Expect: ffmpeg version 7.x
```

### Fix — Option 4: Install recordable as a devDependency

If using `pnpx recordable` (pnpm dlx), the postinstall runs in a throwaway cache that may not persist. Install as a real devDep:

```bash
pnpm add -D -w recordable
# Then in package.json:
#   "scripts": { "rec:demo": "recordable demos/demo.json" }
pnpm rec:demo
```

### Why `pause` hangs after ENOENT

The `pause` action sends an IPC signal to the ffmpeg child process. If ffmpeg never spawned (ENOENT), the signal can't be delivered. recordable blocks waiting for the ack. The script hangs at the first `pause` after the ENOENT.

**Fix:** Fix ffmpeg first (options above). Then the `pause`/`resume` cycle works normally.

## Could not find target

### Symptom

```
[Recordable] RecordableError: Could not find target: "button:has-text(\"Export backup\")"
```

### Cause 1: Playwright-only selector

`button:has-text(...)` is **Playwright-specific**. Recordable uses **Puppeteer**, which doesn't understand `:has-text()`. The selector is passed through as-is, Puppeteer tries to parse it as CSS, sees `:has-text` as an unknown CSS pseudo-class, and returns "not found".

**Fix:** Use `[data-testid='...']` (add the attribute to the button in the app code) or `text:foo` if the text is unambiguous.

### Cause 2: Element doesn't exist yet (race condition)

The element hasn't rendered yet when the action runs.

**Fix:** Add a `waitFor` before the `click`:

```json
{ "action": "waitFor", "target": "text:Copy", "state": "visible", "timeout": 10000 },
{ "action": "click", "target": "text:Copy" }
```

### Cause 3: Element exists but text is different

The button text might be different from what the script expects (e.g. "Save" vs "Save item", or "Export" vs "Export backup").

**Fix:** Open the app in a browser, inspect the actual button text, and use the exact string.

### Cause 4: Element is in a shadow DOM or iframe

Puppeteer can't reach into shadow DOM or iframes with the default selector engine.

**Fix:** Use `>>> ` (shadow piercing) or switch to the iframe first. This is rare for typical SPAs.

## TimeoutError: Waiting for selector failed

### Symptom

```
[Recordable] TimeoutError: Waiting for selector `[[[{"name":"text","value":"Backup created"}]]]` failed
```

### Cause 1: The action that should produce the text never ran

E.g. the script clicked the wrong element (a heading instead of a button), so the success toast never appeared.

**Fix:** Check the recordable log for warnings like `"text:foo matched 2 elements; using the first`. If the first match was a non-clickable element, the click did nothing. Use a unique selector.

### Cause 2: The server is down or returned an error

The click hit the right button, but the server-side operation failed (e.g. 500 error), so the success toast never appeared.

**Fix:** Check the server logs. Look for the API call that the button triggers and verify it returned 200.

### Cause 3: The toast appeared and disappeared too fast

The toast auto-dismissed (e.g. sonner's default 4s) before the `waitFor` polled.

**Fix:** Increase the toast duration in the app, or add a `wait` before `waitFor` to catch the toast earlier.

## Script hangs after `Zoom reset`

### Symptom

The script reaches `Zoom reset` and then stops producing output. No further actions execute.

### Cause

The next action is likely `pause`, and ffmpeg is dead (see "ffmpeg ENOENT" above). The `pause` blocks waiting for ffmpeg to acknowledge.

**Fix:** Fix ffmpeg first. The script will then proceed past `pause`.

## Script bounces to /login after `visit`

### Symptom

```
[Recordable] Visit   http://localhost:5173/dashboard
[Recordable] WaitFor text:Dashboard (visible)
[Recordable] TimeoutError: ...
```

The page bounced to `/login` because the SPA lost the session on reload.

### Cause

`visit` does a full page reload. The app's in-memory auth state is wiped. If the app doesn't restore the session from `IndexedDB`/`localStorage` on mount, the route guard redirects to `/login`.

### Fix

Replace `visit` with `click` on a nav link (client-side navigation — no reload, in-memory state survives):

```json
{ "action": "pause" },
{ "action": "hover", "target": "text:Dashboard" },
{ "action": "click", "target": "text:Dashboard" },
{ "action": "waitFor", "target": "text:Recent projects", "state": "visible", "timeout": 30000 },
{ "action": "resume" }
```

Use a page-unique sentinel (`Recent projects` card title, not `Dashboard` which matches nav + heading).

## Could not launch Chromium

### Symptom

```
[Recordable] Could not launch Chromium: Could not find Chrome (ver. 152.0.7977.75)
```

### Cause

Puppeteer's Chrome browser isn't installed.

### Fix

```bash
pnpx puppeteer browsers install chrome
```

This downloads Chrome to `~/.cache/puppeteer/chrome/`. Verify:

```bash
ls ~/.cache/puppeteer/chrome/
```

## text:foo matched 2 elements; using the first

### Symptom (not an error, but a warning)

```
[Recordable] "text:Dashboard" matched 2 elements; using the first
```

### Cause

The text appears in multiple DOM elements. For `waitFor`/`zoom`, this is fine — any match satisfies the check. For `click`, the first match (in DOM order) is clicked. If the first match is a non-clickable element (e.g. a heading), the click does nothing.

### Fix

- For `waitFor`: usually fine — leave as is.
- For `click`: use a unique selector. Options:
  1. `[data-testid='...']` on the target element (requires app code change)
  2. A more specific CSS selector (e.g. `div.grid > div:first-child button`)
  3. A different text that only appears on the target

## 502 Bad Gateway on fetch

### Symptom

The script clicks a button, but the app shows "502 Bad Gateway" or "ECONNREFUSED" in the browser.

### Cause

The backend server isn't running. The Vite dev server proxies `/api` to `localhost:8080`, but nothing is listening on 8080.

### Fix

Start the backend server:

```bash
# For vautr-server:
VAUTR_DB_URL=sqlite:/tmp/demo.db ./target/debug/vautr-server
```

Verify:

```bash
curl -fsS http://localhost:8080/health
```

## Register fails with 500 (duplicate key)

### Symptom

The script's `register` flow fails with HTTP 500. Server logs show a unique constraint violation.

### Cause

A previous run registered the same username (e.g. `demo@vautr.test`). The DB still has that user. Registering again hits a duplicate-key error.

### Fix

Wipe the DB before each run:

```bash
rm -f /tmp/demo.db /tmp/demo.db-shm /tmp/demo.db-wal
```

Also clear browser site data (DevTools → Application → Storage → Clear site data) so no stale session token interferes.

## General debugging checklist

When a run fails, check in this order:

1. **Is the dev server running?** `curl -fsS http://localhost:5173/` should return HTML
2. **Is the backend running?** `curl -fsS http://localhost:8080/health` should return 200
3. **Is ffmpeg installed?** `ls node_modules/.pnpm/ffmpeg-static@*/node_modules/ffmpeg-static/ffmpeg` should exist
4. **Is Chrome installed?** `ls ~/.cache/puppeteer/chrome/` should have a version dir
5. **Is the DB fresh?** `ls /tmp/demo.db*` — if present, wipe it
6. **Is browser site data cleared?** Open DevTools → Application → Storage → Clear site data
7. **Read the recordable log** — look for "matched 2 elements" warnings, "Could not find target" errors, and "TimeoutError" lines
8. **Read the server log** — look for panics, 500 errors, or missing routes
