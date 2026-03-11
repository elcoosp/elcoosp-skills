# Lingui CLI Workflow

The `@lingui/cli` package provides the engine for the "Extract -> Translate -> Compile" workflow.

## Commands

### `lingui extract`

Scans your source files for macros (`t`, `Trans`, etc.) and generates message catalogs.

```bash
lingui extract
```

````

Options:

- `--overwrite`: Overwrite existing translations for the source locale.
- `--clean`: Remove obsolete messages.

### `lingui compile`

Compiles message catalogs into minified JavaScript files for the runtime.

```bash
lingui compile
```

Options:

- `--strict`: Fail if there are missing translations.

## Setting up npm scripts

Add these to your `package.json`:

```json
{
  "scripts": {
    "extract": "lingui extract",
    "compile": "lingui compile",
    "start": "lingui compile && next start"
  }
}
```

## CI/CD Integration

In a CI environment, you typically want to fail if translations are missing.

```bash
lingui extract --clean && lingui compile --strict
```
````
