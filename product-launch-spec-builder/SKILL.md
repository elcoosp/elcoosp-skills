---
name: product-launch-spec-builder
description: A comprehensive, step‑by‑step workflow to take a new product idea from concept to ready‑to‑implement specifications, including vision, requirements, architecture, UX mockups, technical design, and implementation planning. This skill orchestrates multiple specialised sub‑skills to produce a complete product specification suite.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - AskUserQuestion
  - ExecuteCommand
metadata:
  version: 1.0.0
---

# Product Launch Spec Builder

## When to Use This Skill

Use this skill when you have a **new product idea** and you need to go from a rough concept to a full set of detailed, actionable specifications ready for development. This is ideal for greenfield projects, major features, or product pivots.

**Input:** A brief description of the product idea, target users, and any known constraints.  
**Output:** A complete specification suite including:

1. Product Vision & Strategic Alignment
2. Business & Stakeholder Requirements (BRS)
3. Software Requirements Specification (SRS)
4. Architecture & Design Specification (with ADRs)
5. UX Design Specification (with mockups)
6. Behavioral Specification & Test Verification Plan
7. Technical Design & API Specifications
8. Implementation Plan (with sprint breakdown)
9. Project README

All documents are saved in a structured folder and are ready for stakeholder review or handoff to a development team.

---

## Workflow Overview

The process is divided into **five phases**, each building on the previous. After each phase, you may optionally run a review loop with simulated stakeholders or real feedback.

```
Phase 1: Ideation & Vision
  ├── Use bmad-idea to brainstorm and refine the core concept
  └── Use spec-writer to create the Vision & Strategic Alignment document

Phase 2: Requirements & Domain Modeling
  ├── Use spec-writer to create Business & Stakeholder Requirements (BRS)
  └── Use spec-writer to create Software Requirements Specification (SRS)

Phase 3: Architecture & Design
  ├── Use spec-writer to create Architecture & Design Specification (with ADRs)
  ├── Use spec-writer to create UX Design Specification
  └── Use mockup generator to produce HTML/CSS mockups

Phase 4: Testing & Implementation Planning
  ├── Use spec-writer to create Behavioral Specification & Test Verification Plan
  ├── Use writing-plans to create detailed implementation plan (sprints/tasks)
  └── Use create-readme to generate the project README

Phase 5: Review & Refinement
  ├── Simulate stakeholder feedback (or gather real feedback)
  ├── Update all documents based on feedback
  └── Finalize and package all deliverables
```

---

## Detailed Phase Instructions

### Phase 1: Ideation & Vision

**Goal:** Define the product’s purpose, target users, and high‑level success criteria.

1. **Brainstorm with `bmad-idea`**  
   - Use `/cis-brainstorm` (or `bmad-cis-brainstorming`) to generate ideas around your product concept.  
   - Ask for structured ideation techniques (e.g., “Brainstorm features for a tool that helps solo developers generate content from code”).  
   - Save the brainstorming output in `docs/superpowers/brainstorming/`.

2. **Create the Vision Document**  
   - Invoke `/spec-vision` (from `spec-writer` skill).  
   - Answer the guided questions about:
     - Product type, problem statement, target users
     - Goals and non‑goals
     - Desired outcomes and success metrics
     - Strategic constraints
   - The skill will produce a markdown file named `{project-name}-vision.md` in the project folder.

3. **Review & Iterate (optional)**  
   - If you have stakeholders, share the vision document and gather feedback.  
   - Use the `plan-document-reviewer` prompt (from `writing-plans` skill) to check for completeness.  
   - Revise until approved.

### Phase 2: Requirements & Domain Modeling

**Goal:** Translate the vision into concrete business and software requirements.

1. **Business & Stakeholder Requirements (BRS)**  
   - Run `/spec-brs`.  
   - The skill will ask for:
     - Business context, goals, success metrics
     - Business model and processes
     - Business rules and policies
     - Stakeholder map and user classes (personas / JTBD)
     - Glossary / ubiquitous language
     - Conceptual domain model (entities and relationships)
   - Output: `{project-name}-brs.md`.

2. **Software Requirements Specification (SRS)**  
   - Run `/spec-srs`.  
   - It will reference the BRS and ask you to elaborate:
     - Functional capabilities (with EARS syntax, priority, acceptance criteria)
     - Quality / non‑functional requirements (measurable, using ISO 25010)
     - External interfaces and data contracts
     - Constraints, assumptions, and a TBD log
   - Output: `{project-name}-srs.md`.

3. **Review & Iterate** – again, use the `plan-document-reviewer` to validate each chunk.

### Phase 3: Architecture & Design

**Goal:** Design the system’s structure, UX, and create visual mockups.

1. **Architecture & Design Specification**  
   - Run `/spec-architecture`.  
   - Provide information about:
     - Architecturally significant requirements (from SRS)
     - High‑level design (C4 models, data flow, key components)
     - Technology stack choices
     - Architecture Decision Records (ADRs) for key decisions
   - Output: `{project-name}-architecture.md`.

2. **UX Design Specification**  
   - Run `/spec-ux` (if available; otherwise, manually create using the UX design guidelines).  
   - Define user journeys, wireframes, interaction patterns, and visual style.

3. **Generate Mockups**  
   - Use the **mockup generator** skill to create realistic HTML/CSS mockups.  
   - Workflow:
     - Provide the UX design or describe the screens you need (e.g., dashboard, workflow editor, run details).
     - For each mockup, the skill will produce a self‑contained HTML file with device frames and a spec panel.
     - All mockups are saved in a `mockups/` folder with an `index.html` gallery.

4. **Usability Testing (Simulated)**  
   - You can optionally simulate a usability test using personas (from the BRS) and gather feedback.  
   - Update mockups based on the findings.

### Phase 4: Testing & Implementation Planning

**Goal:** Define how you will verify the system and create a detailed execution plan.

1. **Behavioral Specification & Test Verification**  
   - Run `/spec-test`.  
   - This will guide you through:
     - Writing BDD scenarios (Given/When/Then) for critical features.
     - Defining test strategy (unit, integration, e2e, performance).
     - Creating a Requirements Traceability Matrix (RTM).
     - Outlining NFR verification plans (security, accessibility, etc.).
   - Output: `{project-name}-test-plan.md`.

2. **Implementation Plan**  
   - Use the **writing-plans** skill.  
   - Break down the SRS into user stories and tasks.  
   - Prioritise an MVP scope (must‑have features).  
   - Create a sprint‑by‑sprint plan with detailed tasks (including file paths, code snippets, test commands).  
   - Each task should follow the bite‑sized granularity (2–5 minute steps) and include test steps and commit commands.  
   - Save the plan as `docs/superpowers/plans/YYYY-MM-DD-{feature-name}.md`.

3. **Create the Project README**  
   - Use the **create-readme** skill.  
   - It will scan the project (or you can provide a summary) and produce an appealing `README.md` with:
     - Project title and tagline
     - Key features
     - Quick start instructions
     - Architecture diagram (Mermaid)
     - Tech stack table
     - Configuration notes
     - Usage examples

### Phase 5: Review & Refinement

1. **Stakeholder Review**  
   - Gather all documents and mockups.  
   - Present them to stakeholders (or simulate using personas).  
   - Use the `plan-document-reviewer` prompt for each document to catch issues.

2. **Consolidate Feedback**  
   - Update all affected documents.  
   - Maintain version history (e.g., v0.1 → v0.2).

3. **Final Packaging**  
   - Ensure all files are in a well‑organized folder structure (see below).  
   - Output the final specification suite as a deliverable (e.g., zip archive or push to a repository).

---

## Folder Structure

When following this process, create a project folder with the following structure:

```
project-name/
├── docs/
│   ├── vision.md
│   ├── brs.md
│   ├── srs.md
│   ├── architecture.md
│   ├── ux.md
│   ├── test-plan.md
│   └── superpowers/
│       ├── brainstorming/
│       │   └── brainstorm-{date}.md
│       └── plans/
│           └── YYYY-MM-DD-implementation.md
├── mockups/
│   ├── index.html
│   ├── css/
│   │   └── style.css
│   └── mockups/
│       ├── hero-desktop-mobile.html
│       ├── workflow-editor.html
│       └── ...
├── README.md
└── .env.example
```

---

## Required Sub‑skills

This meta‑skill relies on the following sub‑skills. Make sure they are installed:

- `bmad-idea` – for brainstorming and creative exploration.
- `spec-writer` – for generating the five core specification documents.
- `mockup generator` – for creating HTML/CSS mockups.
- `writing-plans` – for creating detailed implementation plans.
- `create-readme` – for generating the final README.
- `plan-document-reviewer` – for reviewing each chunk (part of `writing-plans`).

If any are missing, you will be prompted to install them.

---

## Quick Start Example

```bash
# 1. Create a project folder
mkdir my-awesome-product
cd my-awesome-product

# 2. Start the product-launch-spec-builder
# (This will guide you through all phases interactively)
product-launch-spec-builder

# 3. Follow the prompts – you'll be asked for your idea and preferences.
```

---

## Tips & Best Practices

- **Keep the user involved:** The process is interactive; ask clarifying questions at each step.
- **Use the `plan-document-reviewer` after each major document** to catch issues early.
- **Version your documents** as you iterate (e.g., v0.1, v0.2, v1.0).
- **For the implementation plan**, remember the “bite‑sized task granularity”: each step should be a single action (2–5 minutes) with a test step and commit.
- **When generating mockups**, always include a specification panel (click “ℹ️ Spec”) so that stakeholders understand the context.
- **After creating the README**, check that the architecture diagram renders correctly on GitHub.

---

## Troubleshooting

- **If a sub‑skill is missing**, the meta‑skill will warn you and suggest the installation command.
- **If you get stuck in a review loop**, consider surfacing the issue to a human for guidance.
- **For very large projects**, you may need to split the process across multiple sessions. The skill will remember progress via the files already created.

---

This meta‑skill is designed to make the journey from idea to implementation smooth, repeatable, and professional. Happy building!
