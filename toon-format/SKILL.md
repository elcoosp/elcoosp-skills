---
name: toon-format
description: |
  Token-Oriented Object Notation (TOON) is a compact encoding of JSON for LLM contexts, achieving 30-60% token savings for uniform arrays while maintaining comparable retrieval accuracy. Use this skill when: (1) Encoding JSON data to TOON format for LLM input, (2) Decoding TOON back to JSON, (3) Explaining or working with TOON syntax, (4) Optimizing token usage for structured data in prompts, (5) Converting between formats (JSON ↔ TOON), (6) Answering questions about TOON specification, benchmarks, or best practices. TOON excels with uniform arrays of objects by declaring fields once and streaming rows compactly.
---

# Toon format

## What is TOON

TOON (Token-Oriented Object Notation) is a compact, human-readable encoding of the JSON data model that minimizes tokens for LLM input. It combines YAML-like indentation for nested objects with CSV-style tabular layout for uniform arrays.

**Core pattern:** Declare structure once, stream data compactly.

```yaml
users[2]{id,name,role}: 1,Alice,admin
  2,Bob,user
```

## When to Use vs. Avoid

**Use TOON for:**

- Uniform arrays of objects (same fields, primitive values) — optimal case
- LLM prompts with structured data where token efficiency matters
- Data requiring structural validation (explicit `[N]` lengths detect truncation)
- Mixed structures with nested objects + tabular arrays

**Avoid TOON for:**

- Deeply nested configs (tabular eligibility ≈ 0%) → use JSON-compact
- Semi-uniform arrays (~40-60% tabular) → use JSON-compact
- Pure tabular data → use CSV (smaller for flat tables)
- Arrays of arrays → use JSON-compact (TOON adds per-array overhead)

**Benchmarks:** TOON achieves 76.4% accuracy vs JSON's 75.0% while using 39.9% fewer tokens on mixed-structure datasets (4 models, 209 questions).

## Complete Syntax Reference

### Objects

Simple objects use `key: value` syntax, one field per line:

```yaml
id: 123
name: Ada
active: true
```

Nested objects add indentation (default: 2 spaces):

```yaml
user:
  id: 123
  name: Ada
```

Empty nested object: `key:` with no children. Empty root object: empty document.

### Arrays

**Primitive arrays (inline):**

```yaml
tags[3]: admin,ops,dev
```

**Tabular arrays (optimal):** When all objects have identical fields with primitive values:

```yaml
items[2]{sku,qty,price}: A1,2,9.99
  B2,1,14.5
```

Header declares: `[N]` length, `{fields}` columns, delimiter (comma default).

**Mixed/non-uniform arrays (list format):**

```yaml
items[3]:
  - 1
  - a: 1
  - text
```

**Objects as list items:**

```yaml
items[2]:
  - id: 1
    name: First
  - id: 2
    name: Second
```

**Tabular array as first field of list-item object:**

```yaml
items[1]:
  - users[2]{id,name}: 1,Ada
      2,Bob
    status: active
```

**Arrays of arrays:**

```yaml
pairs[2]:
  - [2]: 1,2
  - [2]: 3,4
```

**Empty arrays:**

```yaml
items[0]:
```

### Array Headers

Syntax: `key[N<delimiter?>]{fields}:`

- `N` — non-negative integer length
- `delimiter` (optional): absent = comma, `\t` = tab, `|` = pipe
- `fields` (optional for tabular): `{field1,field2,...}`

```yaml
# Comma (default)
items[2]{sku,name}: A1,Widget
  B2,Gadget

# Tab
items[2	]{sku	name}: A1	Widget
  B2	Gadget

# Pipe
items[2|]{sku|name}: A1|Widget
  B2|Gadget
```

**Tip:** Tab delimiters often tokenize more efficiently.

### Key Folding (Optional)

Collapse single-key wrapper chains (spec v1.5+):

```yaml
# Without folding
data:
  metadata:
    items[2]: a,b

# With keyFolding: 'safe'
data.metadata.items[2]: a,b
```

Applies when: each object in chain has exactly one key, leaf is primitive/array/empty object, all segments match `^[A-Za-z_][A-Za-z0-9_]*$`, no collision with existing keys.

Round-trip: encode with `keyFolding: 'safe'`, decode with `expandPaths: 'safe'`.

## Quoting Rules

Strings **must** be quoted if they:

| Condition                          | Example                             |
| ---------------------------------- | ----------------------------------- |
| Empty                              | `""`                                |
| Leading/trailing whitespace        | `" padded "`                        |
| Equals `true`, `false`, or `null`  | `"true"`                            |
| Looks like a number                | `"42"`, `"-3.14"`, `"1e-6"`, `"05"` |
| Contains `:`                       | `"key: value"`                      |
| Contains `"` or `\`                | `"say \"hi\""`                      |
| Contains brackets/braces           | `"[item]"`                          |
| Contains control characters        | `"line1\nline2"`                    |
| Contains active delimiter          | `"hello, world"` (comma delimiter)  |
| Equals `"-"` or starts with `"- "` | `"-"`, `"- text"`                   |

**Unicode and emoji are safe unquoted:**

```yaml
message: Hello 世界 👋
note: This has inner spaces
```

### Escape Sequences

Only 5 valid escapes in quoted strings:

| Character       | Escape |
| --------------- | ------ |
| `\`             | `\\`   |
| `"`             | `\"`   |
| Newline         | `\n`   |
| Carriage return | `\r`   |
| Tab             | `\t`   |

All other escapes (`\x`, `\u`, etc.) are invalid.

## Type Conversions

| Input                             | Output                                             |
| --------------------------------- | -------------------------------------------------- |
| Finite number                     | Canonical decimal (no exponent, no trailing zeros) |
| `NaN`, `Infinity`, `-Infinity`    | `null`                                             |
| `BigInt` (safe range)             | Number                                             |
| `BigInt` (out of range)           | Quoted decimal string                              |
| `Date`                            | ISO string (quoted)                                |
| `undefined`, `function`, `symbol` | `null`                                             |
| `-0`                              | `0`                                                |

Objects with `toJSON()` method: serialized by calling the method, then normalizing result.

## Encoding & Decoding APIs

### TypeScript (`@toon-format/toon`)

```ts
import {
  encode,
  decode,
  encodeLines,
  decodeFromLines,
  decodeStream,
} from "@toon-format/toon";

// Basic
const toon = encode(data, { delimiter: "\t", keyFolding: "safe" });
const data = decode(toon, { strict: true, expandPaths: "safe" });

// Streaming (memory-efficient for large data)
for (const line of encodeLines(largeData)) {
  process.stdout.write(`${line}\n`);
}

const data = decodeFromLines(lines);
for await (const event of decodeStream(asyncLines)) {
  // event: startObject, endObject, startArray, endArray, key, primitive
}
```

**EncodeOptions:**

| Option         | Type                  | Default    | Description             |
| -------------- | --------------------- | ---------- | ----------------------- |
| `indent`       | `number`              | `2`        | Spaces per level        |
| `delimiter`    | `',' \| '\t' \| '\|'` | `','`      | Array delimiter         |
| `keyFolding`   | `'off' \| 'safe'`     | `'off'`    | Collapse wrapper chains |
| `flattenDepth` | `number`              | `Infinity` | Max segments to fold    |
| `replacer`     | `function`            | —          | Transform/filter values |

**DecodeOptions:**

| Option        | Type              | Default | Description                           |
| ------------- | ----------------- | ------- | ------------------------------------- |
| `indent`      | `number`          | `2`     | Expected indentation                  |
| `strict`      | `boolean`         | `true`  | Validate counts, indentation, escapes |
| `expandPaths` | `'off' \| 'safe'` | `'off'` | Reconstruct folded keys               |

**Replacer function:**

```ts
const safe = encode(user, {
  replacer: (key, value, path) => {
    if (key === "password") return undefined; // omit
    if (typeof value === "string") return value.toUpperCase();
    return value;
  },
});
```

### Rust (`toon` crate)

```rust
use toon::{encode, EncodeOptions, Delimiter};
use serde_json::json;

let data = json!({
    "items": [
        { "sku": "A1", "qty": 2 },
        { "sku": "B2", "qty": 1 }
    ]
});

let toon = encode(&data, None);
// items[2]{qty,sku}:
//   2,A1
//   1,B2

// Custom options
let mut opts = EncodeOptions::default();
opts.delimiter = Delimiter::Tab;
opts.length_marker = Some('#');  // items[#2]{...}
let toon = encode(&data, Some(opts));
```

**Cargo.toml:**

```toml
[dependencies]
toon = "0.1"
serde_json = "1.0"
```

**EncodeOptions:**

- `indent: usize` (default: 2)
- `delimiter: Delimiter` (Comma, Tab, Pipe)
- `length_marker: Option<char>` (e.g., `Some('#')`)

### Python (`toon-pyrs`)

```python
import json
import toon

data = {"users": [{"id": 1, "name": "Alice"}]}
result = toon.encode(json.dumps(data), indent=2)
# users[1]{id,name}:
#   1,Alice
```

### CLI (`@toon-format/cli`)

```bash
# Auto-detect by extension
npx @toon-format/cli input.json -o output.toon
npx @toon-format/cli data.toon -o output.json

# Stdin
cat data.json | npx @toon-format/cli --delimiter "\t" --stats

# Options
npx @toon-format/cli input.json \
  --delimiter "\t" \
  --keyFolding safe \
  --stats \
  -o output.toon

# Decode with expansion
npx @toon-format/cli folded.toon --expandPaths safe -o output.json
```

**Key options:**

| Option               | Description                      |
| -------------------- | -------------------------------- |
| `--delimiter <char>` | `,`, `\t`, or `\|`               |
| `--stats`            | Show token savings (encode only) |
| `--keyFolding safe`  | Collapse wrapper chains          |
| `--expandPaths safe` | Reconstruct folded keys          |
| `--no-strict`        | Lenient decode                   |

## LLM Integration

### Sending TOON as Input

Show the format—models parse it naturally:

````
Data in TOON format (2-space indent, arrays show length and fields):

```toon
users[3]{id,name,role}:
  1,Alice,admin
  2,Bob,user
  3,Charlie,user
```

Task: Summarize the user roles.
````

### Generating TOON from LLMs

Show the expected header template:

```
Return filtered data as TOON. Use format: `key[N]{fields}:`
Set [N] to match row count. Output only the code block.
```

### Validate Output

Always decode with strict mode to catch errors:

```ts
try {
  const data = decode(modelOutput, { strict: true });
} catch (error) {
  // Truncation, count mismatch, invalid escapes, etc.
}
```

### Tips

- **Show, don't describe:** A 2-5 row example is more effective than paragraphs
- **Keep examples small:** Models generalize from patterns
- **Always validate:** Don't assume model output is valid TOON
- **Tab delimiters:** Often more token-efficient; specify "tab-separated"

## Implementations

| Language   | Package             | Status                             |
| ---------- | ------------------- | ---------------------------------- |
| TypeScript | `@toon-format/toon` | Reference impl, full encode/decode |
| Rust       | `toon`              | Encode via serde_json              |
| Python     | `toon-pyrs`         | Rust backend via PyO3              |
| Go         | `toon-go`, `gotoon` | Official + community               |
| Java       | `toon-java`         | Official                           |
| Swift      | `toon-swift`        | Official                           |
| .NET       | `toon-dotnet`       | In development                     |
| Others     | See docs            | Community: C++, Ruby, PHP, etc.    |

## Quick Decision Flowchart

```
Is data mostly uniform arrays of objects?
├─ Yes → Use TOON (30-60% token savings)
└─ No → Is it deeply nested or non-uniform?
         ├─ Yes → Use JSON-compact
         └─ No → Is it pure tabular?
                  ├─ Yes → Use CSV
                  └─ No → Use TOON or JSON based on context
```

## Strict Mode Errors

Strict mode (default) catches:

- **Array length mismatch:** `[N]` doesn't match actual row count
- **Delimiter mismatch:** Row delimiters don't match header
- **Indentation errors:** Leading spaces not exact multiples of `indent`
- **Invalid escapes:** `\x`, `\u`, or unterminated strings
- **Syntax errors:** Missing colons, malformed headers
- **Path expansion conflicts:** Overlapping folded keys that can't merge
