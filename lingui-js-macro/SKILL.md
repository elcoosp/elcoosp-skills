---
name: lingui-js-macro
description: Expert guidance for using LinguiJS internationalization (i18n) library with macros. Use when setting up i18n in React, React Native, or JavaScript projects, specifically for extracting messages using the `t`, `plural`, and `Trans` macros. Covers configuration, extraction workflow, and ICU message format patterns.
---

# LinguiJS Macro Skill

This skill provides guidance on implementing internationalization (i18n) in JavaScript/TypeScript projects using LinguiJS macros. It focuses on the compile-time extraction workflow enabled by the macro system (`@lingui/macro`).

## Quick Reference

- [Documentation](https://lingui.dev/)
- [Macro Reference](https://lingui.dev/ref/macro)
- [CLI Reference](https://lingui.dev/ref/cli)
- [Configuration](https://lingui.dev/ref/conf)

## Core Concepts

LinguiJS uses a **macro system** to extract messages at build time. This means you write code that looks like standard function calls or JSX components, but behind the scenes, the macro transforms them into ICU MessageFormat messages and extracts them to a catalog.

### Installation

```bash
npm install @lingui/core @lingui/macro @lingui/cli
# React users should also install:
npm install @lingui/react
```

### Configuration (`lingui.config.ts`)

Create a configuration file in your project root:

```typescript
import { LinguiConfig } from "@lingui/conf";

const config: LinguiConfig = {
  locales: ["en", "es", "fr"],
  sourceLocale: "en",
  catalogs: [
    {
      path: "<rootDir>/src/locales/{locale}/messages",
      include: ["src/"],
    },
  ],
  format: "po", // or 'json', 'po-gettext'
};

export default config;
```

## Macro Usage

### 1. The `t` Macro (Tagged Template Literal)

Used for simple string translation in vanilla JS or logic-heavy areas.

```javascript
import { t } from "@lingui/macro";

function welcome(name) {
  // Variables are automatically extracted as placeholders {name}
  return t`Hello ${name}, welcome to the app!`;
}
```

**Output (Extracted):**

```po
msgid "Hello {name}, welcome to the app!"
msgstr ""
```

### 2. React `<Trans>` Component

Used for translating JSX content. It handles React components inside the translation seamlessly.

```jsx
import { Trans } from "@lingui/react/macro";

function App() {
  return (
    <div>
      <Trans id="welcome.header">Welcome to the application</Trans>
      <Trans>
        Read the <a href="/docs">documentation</a> for more info.
      </Trans>
    </div>
  );
}
```

**Output (Extracted):**

```po
msgid "Welcome to the application"
msgstr ""

msgid "Read the <0>documentation</0> for more info."
msgstr ""
```

### 3. Plurals and Selects

Lingui macros handle ICU plural rules.

```javascript
import { plural, select, selectOrdinal, t } from "@lingui/macro";

function ItemCount({ count }) {
  return plural(count, {
    one: "You have # item",
    other: "You have # items",
  });
}

function GenderSelect({ userGender }) {
  return select(userGender, {
    male: "He is the author",
    female: "She is the author",
    other: "They are the author",
  });
}
```

**Note:** `#` is a placeholder for the count value in plural strings.

### 4. Defining Messages Separately

Use `defineMessage` to define a translation ID without immediately rendering it.

```javascript
import { defineMessage } from "@lingui/macro";

const messages = {
  welcome: defineMessage({
    id: "msg.welcome",
    message: "Welcome!",
    comment: "Header welcome message",
  }),
};

// Access later via the i18n core instance
// i18n._(messages.welcome)
```

## Workflow

Lingui relies on a strict workflow to keep translation files in sync.

1.  **Extract:** Parse source code to generate message catalogs.

    ```bash
    lingui extract
    # or npm run extract
    ```

    This creates/updates `.po` or `.json` files in your locales folder. Files marked as obsolete will be commented out or removed based on config.

2.  **Translate:** Edit the generated `.po` files.

    ```po
    #: src/App.js:10
    msgid "Welcome"
    msgstr "Bienvenue"  # Translate here
    ```

3.  **Compile:** Convert message catalogs into runtime-ready JavaScript/JSON.
    ```bash
    lingui compile
    ```
    This generates `.json` or `.js` files that the runtime `i18n` instance loads.

## Common Pitfalls (Anti-Patterns)

### 1. Forgetting to Import Macros

You must import from `@lingui/macro`. If you forget, the function `t` or component `Trans` will be undefined or not work correctly (no extraction).

```javascript
// WRONG
// import { t } from '@lingui/core'; <-- This is the runtime, NOT the macro!

// CORRECT
import { t } from "@lingui/macro";
```

### 2. String Concatenation in Plurals

Do not concatenate strings inside macros. It breaks extraction.

```javascript
// WRONG
plural(count, {
  one: "You have " + count + " item", // Macro cannot extract this correctly
  other: "You have " + count + " items",
});

// CORRECT
plural(count, {
  one: "You have # item",
  other: "You have # items",
});
```

### 3. Dynamic IDs

Message IDs should be static strings.

```javascript
// WRONG
const id = "msg." + type;
<Trans id={id}>...</Trans>

// CORRECT
<Trans id="msg.typeA">...</Trans>
```

### 4. Not Compiling Catalogs

If translations appear as keys (IDs) instead of the translated strings, you likely forgot to run `lingui compile` or load the catalog in your app initialization.

## Initialization (Runtime)

Macros handle extraction, but you need to load the compiled catalogs at runtime using `@lingui/core`.

```javascript
import { i18n } from "@lingui/core";
import { messages as enMessages } from "./locales/en/messages.js";
import { messages as esMessages } from "./locales/es/messages.js";

i18n.load("en", enMessages);
i18n.load("es", esMessages);
i18n.activate("en"); // Set active locale
```

---

### references/configuration.md

- Essential configuration properties (`locales`, `sourceLocale`, `catalogs`, `format`)
- Catalog path configuration and source mapping
- Supported catalog formats (`po`, `json`, `po-gettext`) and their trade-offs
- Extractor configuration for Vue and custom file types
- Runtime configuration module setup

### references/react-integration.md

- Usage of the `<Trans>` component for JSX translation
- Differences between `t` macro and `<Trans>` component
- `useLingui` macro for dynamic translations inside components
- Plural and Select components in React context

### references/cli-workflow.md

- Core CLI commands: `extract`, `compile`
- Extraction options (`--overwrite`, `--clean`)
- Compilation options (`--strict` for CI/CD)
- Recommended npm scripts setup
- CI/CD integration patterns

This skill provides comprehensive coverage of LinguiJS macros, focusing on correct setup, extraction workflows, and ICU message format patterns to avoid common internationalization pitfalls.
