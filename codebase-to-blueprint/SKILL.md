---
name: codebase-to-blueprint
description: >
  Performs an exhaustive, language-agnostic analysis of any codebase and produces a single
  self-contained BLUEPRINT.md file. Use this skill whenever the user wants to document,
  reverse-engineer, or create a reconstruction guide for a codebase — regardless of stack,
  framework, or architecture. Trigger for phrases like: "analyse my codebase", "document
  this project", "create a blueprint", "reverse engineer this repo", "generate a BLUEPRINT.md",
  "make this project reproducible", "document my code for an AI agent", or "I want an AI to
  be able to recreate this project". Also trigger when the user uploads or points to a project
  directory and asks for comprehensive documentation. The output MUST be 100% self-contained
  and independent — if handed to a completely fresh AI agent with zero additional context, that
  agent must be able to reconstruct the entire project from scratch: every file, all logic,
  exact structure, correct wiring, and a working Mermaid architecture flowchart.
---

# Codebase to Blueprint

Produce a single `BLUEPRINT.md` that is **100% self-contained and context-independent**.

> **The Prime Directive:** A fresh AI agent handed ONLY this file — with no access to the
> original repo, no conversation history, no extra context whatsoever — must be able to
> reconstruct the entire project from scratch. Every file, all logic, exact folder structure,
> correct wiring, every environment variable, every dependency version, and every build step
> must be present and unambiguous inside this one document. If any detail requires the reader
> to "look it up" or "infer from context", it is missing and must be added.

---

## Inputs

- **Root path** — provided by the user, or default to the current working directory (`$PWD`).
- Accept any language, framework, or architecture.

---

## Step 0 — Reconnaissance

Before writing anything, map what you're working with:

```bash
# Get directory tree (ignore common noise)
find "$ROOT" \
  -not \( -path "*/node_modules/*" -o -path "*/.git/*" -o -path "*/__pycache__/*" \
          -o -path "*/dist/*" -o -path "*/build/*" -o -path "*/.next/*" \
          -o -path "*/target/*" -o -path "*/venv/*" -o -path "*/.env" \
          -o -name "*.pyc" -o -name "*.class" -o -name "*.o" \) \
  -type f | sort
```

Read these files first (they contain the most orientation value):
- `README.md`, `README.rst`, `README.txt` (any variant)
- `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, `build.gradle`
- `docker-compose.yml`, `Dockerfile`
- `.env.example`, `config/`, `settings.py`, `config.py`
- Any `ARCHITECTURE.md`, `DESIGN.md`, `CONTRIBUTING.md`

Infer the stack before reading individual source files.

---

## Step 1 — Traverse & Read All Files

Recursively read every non-binary, non-ignored file. Apply `.gitignore` rules.

**Always skip:**
`node_modules/`, `.git/`, `__pycache__/`, `dist/`, `build/`, `.next/`, `target/`, `venv/`,
`.env`, `*.pyc`, `*.class`, `*.o`, `*.so`, `*.exe`, `*.dll`, `*.bin`, `*.jpg`, `*.png`,
`*.gif`, `*.ico`, `*.woff`, `*.ttf`, `*.lock` (read, don't embed verbatim).

For **lock files** (`package-lock.json`, `yarn.lock`, `Pipfile.lock`, `poetry.lock`,
`Cargo.lock`): read to extract exact versions; do NOT embed the full file.

---

## Step 2 — Write BLUEPRINT.md

Write the file section by section. Use `##` for top-level sections and `###` for subsections.
All code, configs, types, and schemas go in fenced blocks with the correct language tag.

**Embedding rules:**
- Files **≤ 100 lines**: embed the **full source**.
- Files **> 100 lines**: embed the **most critical sections** (entry points, exported APIs,
  core logic, key types) and summarise omitted parts with a detailed description — detailed
  enough that an AI could reconstruct the omitted logic without the original file.

**Self-containment rules (strictly enforced):**
- Never write "see the source file" or "refer to the repo" — the blueprint IS the source of truth.
- Never omit a function signature, type definition, or config value with a vague placeholder.
- Never use phrases like "standard boilerplate" or "typical setup" — write the actual content.
- Every import path, every exported name, every env var, and every CLI flag must appear explicitly.
- Inline comments in embedded code must be preserved verbatim; they carry intent.

If a section cannot be determined from the codebase, write exactly:
`[NOT DETERMINABLE — requires clarification]`

---

### Required Sections

#### `## 0. Self-Containment Guarantee`
Open the blueprint with this boilerplate block (fill in the project name):

```
This document is the single source of truth for reconstructing [PROJECT NAME].
It is 100% self-contained. An AI agent with access to only this file and a
standard development environment can recreate the entire project without any
additional context, repositories, conversations, or external references.
```

Then list the exact runtime versions required (Node 20.x, Python 3.11, Go 1.22, etc.)
so the reconstructing agent can set up an identical environment.

---

#### `## 1. Project Identity`
- Name, one-sentence purpose, and extended description
- Tech stack summary: languages, frameworks, runtimes, major libraries with versions
- Target audience and primary use case

#### `## 2. Repository Structure`
Annotated directory tree. Every directory and key file gets a one-line role description.

```
root/
├── src/           # Application source code
│   ├── api/       # Express route handlers
│   └── models/    # Mongoose schemas
├── tests/         # Jest test suites
└── package.json   # Project manifest and scripts
```

#### `## 3. Architecture`

This section has TWO required parts:

**Part A — Prose & ASCII Diagram**
- Architectural pattern (MVC, layered, microservices, feature-based, monorepo, etc.)
- How major parts connect — prose narrative **plus** ASCII diagram
- Key design decisions inferred from code when not documented

**Part B — Mermaid Architecture Flowchart (REQUIRED)**

Produce a detailed Mermaid `flowchart LR` (or `flowchart TD` if it fits better) that captures:
- Every major module, service, layer, or package as a labelled node
- Every significant data or control flow as a directed edge with a label describing what flows
- External dependencies (DBs, APIs, queues, caches) as distinctly styled nodes
- Entry points (HTTP, CLI, cron, event) clearly marked
- Auth boundaries, middleware chains, and error paths if present

Use Mermaid subgraphs to group related nodes (e.g. `subgraph Frontend`, `subgraph Backend`).
Apply node styles to distinguish layers using a `classDef` block.

Minimum fidelity: a reader must be able to trace any request or data event from its origin
to its final destination purely from the diagram.

Example structure (replace with actual project content):

```mermaid
flowchart LR
  classDef ui fill:#4A90D9,stroke:#2C5F8A,color:#fff
  classDef api fill:#27AE60,stroke:#1A7A42,color:#fff
  classDef db fill:#8E44AD,stroke:#6C2D8A,color:#fff
  classDef ext fill:#E67E22,stroke:#B85C10,color:#fff

  Client([Browser / Mobile]):::ui
  subgraph Backend
    Router[API Router]:::api
    Auth[Auth Middleware]:::api
    Handler[Route Handlers]:::api
    Service[Business Logic]:::api
  end
  DB[(PostgreSQL)]:::db
  Cache[(Redis)]:::db
  ExtAPI[Third-party API]:::ext

  Client -->|"HTTPS request"| Router
  Router -->|"verify token"| Auth
  Auth -->|"authorised"| Handler
  Handler -->|"call"| Service
  Service -->|"query"| DB
  Service -->|"cache read/write"| Cache
  Service -->|"HTTP call"| ExtAPI
```

If the project is too simple for a meaningful flowchart, produce a minimal one and note why.
**Never omit this section** — even a trivial three-node diagram is required.

#### `## 4. File Inventory`
One subsection per file (or per logical group for large projects). For each file:
- **Path** and **purpose**
- **Full source** if ≤ 100 lines; critical excerpts + detailed prose summary if > 100 lines
- **Exports**: functions, classes, components, routes, middleware — with full signatures,
  parameter types, return types, and one-line behaviour descriptions
- **Imports / internal deps** — every import listed explicitly
- Flag: `⚠ CRITICAL BUSINESS LOGIC` if applicable

#### `## 5. Data Models & Schema`
- All types, interfaces, DB schemas, ORM models — full definitions, no abbreviations
- Entity relationships, validation rules, constraints
- SQL: full `CREATE TABLE` DDL with indexes and foreign keys. NoSQL: document shape with
  field types, required vs optional, and example values.

#### `## 6. API & Integration Layer`
For every endpoint:
```
METHOD /path
  Auth:     <required|none|<scheme>>
  Request:  <body shape or params — full field list with types>
  Response: <shape + status codes — full field list with types>
  Notes:    <any special behaviour, rate limits, side effects>
```
- Internal service-to-service calls with full request/response shapes
- Third-party SDKs: which ones, how authenticated, how called, which env vars they consume
- WebSocket / queue / event-driven interfaces: event names, payload shapes

#### `## 7. Configuration & Environment`
| Variable | Type | Required | Default | Example | Description |
|----------|------|----------|---------|---------|-------------|
| `DATABASE_URL` | string | yes | — | `postgres://user:pass@localhost:5432/db` | Primary DB connection |

- Every variable that appears anywhere in the codebase must be listed.
- Embed all config files (e.g. `tsconfig.json`, `vite.config.ts`, `.eslintrc`) in full if ≤ 100 lines.

#### `## 8. State Management`
*(Skip if not applicable — mark with `N/A`)*
- Client-side: stores, contexts, signals — their shape, initial state, and all update patterns
- Server-side: session strategy, cache layer, shared state

#### `## 9. Dependencies`
| Package | Exact Version | Type | Role |
|---------|--------------|------|------|
| `express` | `4.18.2` | runtime | HTTP server framework |
| `jest` | `29.0.0` | dev | Test runner |

- Use exact versions from the lockfile, not semver ranges.
- Flag unusual or pinned version constraints and explain why they are pinned.

#### `## 10. Tooling & Build System`
- Package manager and lockfile
- Build / bundler config (embed key config files in full)
- Compiler / transpiler settings
- Linting, formatting, testing config
- All runnable scripts with full descriptions and expected output:

```
npm run dev       # Start dev server with hot reload on port 3000
npm run build     # Production build → dist/; outputs index.js + assets
npm test          # Run Jest test suite; requires TEST_DB_URL env var
```
- CI/CD: embed all relevant workflow files in full

#### `## 11. Patterns & Conventions`
- File, variable, function, route, component naming conventions — with concrete examples
- Recurring code patterns — with representative code snippets
- Error handling strategy — how errors are caught, wrapped, logged, and surfaced
- Auth/authorisation approach (JWT, sessions, OAuth, RBAC, etc.) — full flow described
- Logging and observability — what is logged, at what level, in what format

#### `## 12. Reproduction Instructions`

> **These instructions must be executable by a developer on a clean machine with no prior
> knowledge of this project.** Every command must be exact. Every prerequisite must be listed.

Step-by-step from zero to running:

```bash
# 1. Install runtime (specify exact version)
# 2. Install dependencies
# 3. Configure environment (list every required .env value)
# 4. Database setup / migrations / seed
# 5. Start the application
# 6. Verify it is working (health check URL, expected output, etc.)
```

- List every one-time setup step: DB creation, secret generation, third-party account setup.
- Write commands for the **detected** stack — do not use npm commands for a Python project.
- If there are multiple services (frontend + backend + worker), provide commands for each.
- Include the expected output of the final step so the developer knows it worked.

---

## Step 3 — Quality Check Before Saving

Before writing the file, run through every item in this checklist. **All must pass.**

**Completeness**
- [ ] Every source file is accounted for in §4 (File Inventory)
- [ ] Every environment variable from the codebase appears in §7
- [ ] Every dependency from every manifest appears in §9 with an exact version
- [ ] Reproduction instructions in §12 cover every prerequisite and produce a running app

**Self-Containment (Critical)**
- [ ] A fresh AI agent given ONLY this file could write every source file without guessing
- [ ] No section says "see the source", "refer to the repo", or "standard boilerplate"
- [ ] No function, type, or config value is described with a vague placeholder
- [ ] Every import path referenced in §4 is traceable to a file documented in §4
- [ ] The Mermaid diagram in §3 renders valid syntax and covers all major components
- [ ] §0 self-containment guarantee block is present and filled in

**Formatting**
- [ ] All code blocks have the correct language tag
- [ ] The Mermaid block uses valid Mermaid syntax (no unsupported node shapes or keywords)

---

## Step 4 — Save

Save as `BLUEPRINT.md` in the root of the analysed directory. Report the output path and
file size to the user.

---

## Edge Cases

| Situation | Handling |
|-----------|----------|
| Monorepo | Document each package/app in its own §4 subsection; one shared §3 Mermaid diagram covering all packages and their inter-dependencies |
| Microservices | One §6 section per service; Mermaid diagram must show all services and inter-service message contracts |
| Generated files | Note they are generated; document the generator config and the generation command instead of the generated output |
| Binary assets | List them in §2 with a brief description and their role; do not embed |
| Encrypted / secret files | Note existence and purpose; describe the expected format; never embed actual secrets |
| Missing docs / no README | Infer everything from code; mark every inference with `[INFERRED]`; produce the Mermaid diagram from code structure |
| Empty sections | Always write the heading; use `N/A` or `[NOT DETERMINABLE — requires clarification]` |
| Very small project (< 5 files) | Still produce all sections; sections that don't apply get `N/A`; Mermaid diagram still required, even if minimal |
| Very large project (> 100 files) | Group files into logical modules in §4; ensure the Mermaid diagram uses subgraphs to remain readable |