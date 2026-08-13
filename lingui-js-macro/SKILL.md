---
name: Lingui 6 i18n Expert
description: Comprehensive guide for AI agents on using Lingui 6 internationalization – with a strong focus on macros as the primary developer experience for writing translations in JavaScript/TypeScript, React, React Native, and Node.js.
---

## Skill Name: Lingui 6 i18n Expert

**Purpose:**  
Provide accurate, idiomatic guidance on using Lingui 6, with a special focus on **macros** – the primary developer experience (DX) tool for writing translations. Macros are compile‑time transforms that let you write natural JS/JSX and automatically generate ICU MessageFormat catalogs.

---

### 1. Core Principles & Workflow

1. **Define Messages** – Use **macros** (`Trans`, `t`, `msg`, `Plural`, `Select`) directly in your source code.
2. **Extract** – Run `lingui extract` to collect all messages into catalog files.
3. **Translate** – Hand off catalogs to translators (or use a TMS).
4. **Compile** – Run `lingui compile` to produce lean, runtime‑optimized JavaScript.
5. **Deploy** – Ship compiled catalogs with your app.

---

### 2. Macro Setup (Transpiler Configuration)

Macros are **not** runtime functions – they are transformed at build time. You **must** configure your transpiler to process them.

#### Babel
- Install `@lingui/babel-plugin-lingui-macro`.
- Add it to your Babel config (`.babelrc` or `babel.config.js`).
```json
{
  "plugins": ["@lingui/babel-plugin-lingui-macro"]
}
```
- If using Vite with `@vitejs/plugin-react`, set `babel: { plugins: ["@lingui/babel-plugin-lingui-macro"] }`.

#### SWC
- Install `@lingui/swc-plugin`.
- In `.swcrc` or `next.config.js`, add `"@lingui/swc-plugin"` to `jsc.experimental.plugins` (or `experimental.swcPlugins` in Next.js).
```js
// next.config.js
experimental: {
  swcPlugins: [
    ["@lingui/swc-plugin", { /* options */ }]
  ]
}
```

#### Vite (with Rolldown/Babel)
- For Vite 8+ using Rolldown, you can use `@rolldown/plugin-babel` along with the `linguiTransformerBabelPreset` exported from `@lingui/vite-plugin`.

**Important:** Without this setup, macros will throw runtime errors (they are designed to fail if not transpiled).

---

### 3. Using Macros – The Preferred DX

All macros are imported from dedicated entry points:
- **Core (JS) macros** → `@lingui/core/macro`
- **React (JSX) macros** → `@lingui/react/macro`

#### Core Macros (for any JavaScript context)

- **`t`** – Tagged template literal for immediate translation.
```js
import { t } from "@lingui/core/macro";
const msg = t`Hello ${user.name}`;
```
- **`msg`** – Defines a message descriptor for later use (lazy translation).
```js
import { msg } from "@lingui/core/macro";
const welcome = msg`Welcome!`;
// later: i18n._(welcome)
```
- **`plural`** – Handles plural forms.
```js
const countLabel = plural(count, {
  one: "# book",
  other: "# books",
});
```
- **`select`** – Selects a case based on a value.
```js
const label = select(gender, {
  male: "He",
  female: "She",
  other: "They",
});
```
- **`ph`** – **Explicit placeholder naming** (critical for translator context). Use it to give meaningful names to expressions that would otherwise become `{0}`.
```js
t`Hello ${ph({ name: getUserName() })}`; // => "Hello {name}"
```

#### React JSX Macros (for React components)

- **`<Trans>`** – The workhorse for translating JSX, including rich text and nested components.
```jsx
import { Trans } from "@lingui/react/macro";
<Trans>Welcome back, {userName}!</Trans>
<Trans>Read <a href="/docs">the docs</a>.</Trans>
```
- **`<Plural>`** – Pluralization inside JSX.
```jsx
<Plural value={count} one="# item" other="# items" />
```
- **`<Select>`** – Switch between variants.
```jsx
<Select value={gender} _male="He" _female="She" other="They" />
```
- **`useLingui`** – Provides a **context‑aware `t` function** for translations outside JSX (e.g., props, alerts). This is a macro that transpiles to the runtime `useLingui` hook.
```jsx
const { t } = useLingui();
return <button aria-label={t`Close`}>Close</button>;
```

---

### 4. Macros vs. Runtime Components

- Macros are **compile‑time**. They produce the `id` and `message` props for the runtime components (`Trans`, `Plural`, etc.) and strip away non‑essential fields (`comment`, `message`) in production.
- The runtime components are imported from `@lingui/react` (or `@lingui/solid`) and are what actually render translations.
- **Always prefer macros** – they reduce boilerplate, enforce best practices, and keep your code clean.

---

### 5. The `I18nProvider` and Catalog Loading

- Wrap your app with `<I18nProvider i18n={i18n}>`.
- Load catalogs dynamically:
```ts
import { i18n } from "@lingui/core";

export async function dynamicActivate(locale: string) {
  const { messages } = await import(`./locales/${locale}/messages`);
  i18n.loadAndActivate({ locale, messages });
}
```
- Use `fallbackLocales` in config to handle missing translations.

---

### 6. CLI Commands (Extract & Compile)

- `lingui extract` – Scans source files, extracts messages, updates catalogs.
- `lingui extract --clean` – Removes obsolete messages.
- `lingui compile` – Generates runtime‑optimized JS/TS files.
- `lingui compile --typescript` – Produces `.ts` catalogs with type definitions.

---

### 7. Common Pitfalls to Avoid

- **Using macros at module level** – They will be evaluated at import time, before the locale is set. Always use them inside functions or components.
- **Forgetting to run `extract` before building** – Production builds without extracted catalogs will show raw message IDs.
- **Mixing ESM/CommonJS incorrectly** – Lingui 6 is ESM‑only; ensure your project is compatible.
- **Using `t` or `msg` without a transpiler setup** – Macros must be transpiled; otherwise they throw runtime errors.

---

### 8. Framework-Specific Considerations

- **React Server Components (RSC):** Use `setI18n` from `@lingui/react/server` to provide the i18n instance to server components.
- **Next.js:** Use middleware for locale detection and `generateStaticParams` for static generation.
- **React Native:** Use `@lingui/metro-transformer` to compile `.po` files on the fly; set `defaultComponent={Text}`.
- **SolidJS:** Use `@lingui/solid`; access `i18n` as a signal (`i18n()`).

---

### 9. Quick Reference – Where to Import

| Macro / Component            | Import from                      |
|------------------------------|----------------------------------|
| `t`, `msg`, `plural`, `select`, `ph` | `@lingui/core/macro`         |
| `Trans`, `Plural`, `Select`, `useLingui` (macro) | `@lingui/react/macro` |
| `I18nProvider`, `useLingui` (runtime) | `@lingui/react`             |
| `i18n` (instance)            | `@lingui/core`                   |
