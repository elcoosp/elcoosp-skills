# Elcoosp Skills

A curated collection of specialized skills designed to extend AI agent capabilities with domain-specific knowledge, workflows, and tools.

## 🤔 What are Skills?

Skills are modular, self-contained packages that act as "onboarding guides" for specific domains or tasks. They transform a general-purpose AI agent into a specialized expert equipped with procedural knowledge, best practices, and anti-pattern avoidance strategies.

Each skill consists of a main instruction file (`SKILL.md`) and optional reference materials to provide deep context without cluttering the main context window.

For more details on the skills standard, see [agentskills.io](https://agentskills.io/home).

## 📦 Available Skills

### `sea-orm-2`

Expert guidance for **SeaORM 2.0**, Rust's async ORM. This skill is essential for projects using the latest SeaORM version, as it focuses on the new patterns that differ significantly from 1.0.

**Key Features:**

- **New Entity Format:** Covers the `#[sea_orm::model]` macro and defining relations directly on the `Model` struct.
- **Strongly-Typed Columns:** Usage of the `COLUMN` constant for compile-time type safety.
- **Entity Loader API:** Efficient eager loading patterns to eliminate N+1 queries.
- **Nested ActiveModel:** The new builder pattern for persisting complex object graphs.
- **Entity-First Workflow:** Using Schema Registry for automatic schema synchronization.
- **Anti-Patterns:** Critical warnings against common 1.0 habits that are now incorrect.

**Bundled References:**

- `entity-patterns.md`: Detailed examples for relations, composite keys, and custom types.
- `activemodel-patterns.md`: Creation, builder pattern, and nested persistence.
- `query-patterns.md`: Filtering, joins, pagination, and raw SQL.
- `migration-guide.md`: Step-by-step migration from 1.0 to 2.0.

### `lingui-js-macro`

Expert guidance for **LinguiJS**, a readable, automated, and optimized internationalization framework for JavaScript. This skill focuses on the macro system (`@lingui/macro`) which enables compile-time extraction of messages, offering a clean syntax and powerful ICU MessageFormat support.

**Key Features:**

- **Macro System:** Correct usage of `t`, `Trans`, `plural`, and `select` macros for static message extraction.
- **ICU MessageFormat:** Patterns for handling complex plural rules, gender selections, and rich-text translations.
- **Extraction Workflow:** CLI commands (`extract`, `compile`) to keep translation catalogs in sync with source code.
- **React Integration:** Seamless usage of `<Trans>` components and `useLingui` hooks for component-level translation.
- **Configuration:** Setting up `lingui.config.ts` for catalogs, locales, and formats.

**Bundled References:**

- `configuration.md`: Catalog formats, extractor config, and runtime setup.
- `react-integration.md`: JSX translation patterns with `<Trans>`.
- `cli-workflow.md`: Extract, translate, compile workflow for CI/CD.

### `toon-format`

Expert guidance for **TOON (Token-Oriented Object Notation)**, a compact encoding of JSON designed for LLM contexts. This skill helps optimize token usage when passing structured data to models, achieving 30-60% savings for uniform arrays while maintaining retrieval accuracy.

**Key Features:**

- **Token Efficiency:** Strategies to achieve 30-60% token reduction for uniform arrays and ~40% for mixed-structure datasets compared to JSON.
- **Syntax Patterns:** Complete coverage of objects, nested structures, primitive arrays, tabular arrays, and mixed arrays with quoting and escaping rules.
- **LLM Integration:** Best practices for using TOON in prompts (input) and parsing model output (output), including validation with strict mode.
- **Multi-Language APIs:** Usage examples for TypeScript (`@toon-format/toon`), Rust (`toon` crate), Python (`toon-pyrs`), and the CLI tool.
- **Decision Framework:** Clear guidelines on when to use TOON versus JSON-compact or CSV based on data structure (uniformity, nesting depth, tabular eligibility).
- **Strict Validation:** Using array length markers `[N]` and field headers `{fields}` to detect truncation and malformed output.

## 🚀 Getting Started

### Prerequisites

These skills are designed to be used with AI agents that support the "Skills" format (e.g., Codex).

### Installation

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/elcoosp/elcoosp-skills.git
    ```

2.  **Configure your AI agent:**
    Point your agent's configuration to the `skills/` directory or the specific skill path you wish to use.

    _Example configuration:_

    ```yaml
    skills:
      - path: ./elcoosp-skills/sea-orm-2
      - path: ./elcoosp-skills/lingui-js-macro
      - path: ./elcoosp-skills/toon-format
    ```

## 📖 Skill Anatomy

Skills in this repository follow the standard structure:

```
skill-name/
├── SKILL.md              # Main entry point. Contains core instructions and triggers.
└── references/           # Deep-dive documentation loaded as needed.
    ├── pattern-a.md
    └── pattern-b.md
```

- **SKILL.md:** Contains metadata (name, description) that the AI uses to decide _when_ to load the skill, and concise instructions on _how_ to use it.
- **References:** Detailed files that provide extensive examples, API references, and migration guides. These are only loaded when the AI determines they are necessary for the task, preserving context window efficiency.

## 🤝 Contributing

Contributions are welcome! If you have a skill you'd like to add or improvements to existing ones:

1.  Fork the repository.
2.  Create your feature branch (`git checkout -b feature/new-skill`).
3.  Commit your changes (`git commit -m 'Add some skill'`).
4.  Push to the branch (`git push origin feature/new-skill`).
5.  Open a Pull Request.

### Creating a New Skill

Ensure your skill adheres to the following principles:

- **Concise is Key:** Avoid verbose explanations. Prefer examples.
- **Progressive Disclosure:** Keep core logic in `SKILL.md` and deep details in `references/`.
- **Clear Triggers:** The description in the frontmatter must clearly state when the skill should be used.

## 📜 License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.

---

_Built with the Skill Creator pattern._
