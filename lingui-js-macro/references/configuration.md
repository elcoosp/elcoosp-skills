# LinguiJS Configuration Reference

Configuration is typically stored in `lingui.config.js`, `lingui.config.ts`, or the `lingui` section of `package.json`.

## Essential Properties

| Property       | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `locales`      | Array of locale codes (e.g., `['en', 'fr', 'ja']`).          |
| `sourceLocale` | The locale you develop in (usually the first language).      |
| `catalogs`     | Array of objects defining where to store extracted messages. |
| `format`       | Catalog format: `po`, `po-gettext`, `json`, `csv`.           |

## Catalog Configuration

The `catalogs` array maps your source code to translation files.

```typescript
catalogs: [
  {
    path: "<rootDir>/src/locales/{locale}",
    include: ["src/"],
    exclude: ["node_modules/"],
  },
];
```

## Formats

- **`po` (Recommended):** Standard gettext format. Excellent tooling support (Poedit, Crowdin, etc.). Supports plurals natively.
- **`json`:** Simple key-value JSON. Easier to parse manually but lacks native plural support in the format.
- **`po-gettext`:** Standard gettext format with specific plural header handling.

## Advanced: Extractor Configuration

If using Vue or custom file types, configure extractors:

```typescript
extractors: [
  {
    match: "**/*.vue",
    extractor: "@lingui/extractor-vue",
  },
];
```

## Runtime Config Module

You can specify a module to configure the i18n instance automatically:

```typescript
runtimeConfigModule: {
  i18n: ['@lingui/core', 'i18n'],
  Trans: ['@lingui/react', 'Trans']
}
```
