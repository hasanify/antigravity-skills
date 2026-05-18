---
name: readme-sync
description: >
  Analyzes a codebase and automatically generates or updates its README.md to accurately reflect
  the current state of the project, including a detailed embedded Mermaid architecture flowchart.
  Use this skill whenever the user asks to "update my README", "sync my README with the code",
  "generate a README for this project", "my README is outdated", "write documentation for this
  repo", or any similar request involving README generation or documentation that should reflect
  the actual codebase. Trigger even for casual phrasings like "can you fix my README?" or
  "document this project for me". Works for any language, framework, or stack — Python, Node,
  Rust, Go, monorepos, CLI tools, APIs, frontend apps, anything.
---

# readme-sync

Analyzes a codebase and rewrites or creates a single `README.md` that accurately reflects the
current state of the project, including a fully detailed Mermaid architecture diagram embedded
directly in the README. No separate architecture file is created. Language-, framework-, and
stack-agnostic.

---

## Workflow

### Phase 1 — Locate the codebase root

If the user provided a path, use it. Otherwise assume the current working directory. Confirm the
root before proceeding.

```bash
ls -1A <root>
```

---

### Phase 2 — Deep audit

Run all of the following discovery steps. Collect findings into a structured mental model before
touching the README.

#### 2a. Directory structure

```bash
find <root> -maxdepth 3 \
  -not -path '*/.git/*' \
  -not -path '*/node_modules/*' \
  -not -path '*/__pycache__/*' \
  -not -path '*/.venv/*' \
  -not -path '*/vendor/*' \
  -not -path '*/.next/*' \
  -not -path '*/dist/*' \
  -not -path '*/build/*' \
  -not -path '*/target/*' \
  | sort
```

Use the structure to infer: Is this a monorepo? A library? A service? A CLI tool? A fullstack app?

#### 2b. Config and manifest files

Read every file that exists (use `cat` or `view`):

| File | What to extract |
|------|----------------|
| `package.json` | name, description, scripts, dependencies, engines |
| `pyproject.toml` / `setup.py` / `setup.cfg` | name, description, dependencies, entry points |
| `Cargo.toml` | name, description, edition, dependencies, [[bin]] |
| `go.mod` | module path, Go version, require block |
| `pom.xml` / `build.gradle` | artifactId, dependencies, plugins |
| `Makefile` | all targets and their recipes |
| `docker-compose.yml` / `compose.yaml` | services, ports, volumes, dependencies |
| `Dockerfile` | base image, exposed ports, CMD/ENTRYPOINT |
| `.env.example` / `.env.sample` / `.env.template` | all variable names + inline comments |
| `Procfile` | process types and commands |
| `fly.toml` / `railway.toml` / `render.yaml` / `vercel.json` / `netlify.toml` | deploy config, build commands |
| `.github/workflows/*.yml` | CI steps, test commands, deploy targets |
| `turbo.json` / `nx.json` / `lerna.json` | monorepo workspace config |
| `justfile` | recipes and commands |

#### 2c. Entry points and source

Scan top-level source directories (`src/`, `app/`, `lib/`, `cmd/`, `api/`, `packages/`, etc.).
Read enough source to understand the project's purpose — focus on:
- `main.*`, `index.*`, `app.*`, `server.*`, `cli.*`
- Route definitions (express, fastapi, gin, axum, etc.)
- Key exported modules or public API surface

Do **not** read test files for purpose-inference (they confirm what exists, not what it does).

#### 2d. Existing README

If a `README.md` exists, read it fully. Note:
- Sections that are still accurate → **preserve verbatim**
- Sections that are outdated or contradict the code → **rewrite**
- Sections absent from the codebase (e.g. contributing guide, license, philosophy) → **keep as-is**
- Sections missing from the README that the codebase implies → **add**

---

### Phase 3 — Build the mental model

Before writing, synthesize findings into this structure (internal reasoning only, not written out):

```
Project:
  name:
  one-line description:
  type: [library | CLI | API | web app | service | monorepo | ...]

Stack:
  language(s):
  runtime/version:
  framework(s):
  key dependencies:

Entry points:
  dev command:
  prod command:
  test command:
  build command:
  other scripts/targets:

Environment:
  required vars: [name, purpose]
  optional vars: [name, purpose, default]

Structure:
  notable dirs/modules and what they do:

Deploy:
  platform: [if detectable]
  method:

Data flows:
  inputs:        [where data/requests enter the system]
  processing:    [key transformation/business logic steps]
  outputs:       [what the system produces or responds with]
  external deps: [databases, queues, third-party APIs, etc.]
```

If any field cannot be confidently inferred, mark it `UNKNOWN` — these become `<!-- TODO: fill in -->` placeholders.

---

### Phase 4 — Design the Mermaid architecture diagram

Design the full system diagram before writing any README prose. The diagram will be embedded
directly in the README under `## System Architecture & Data Flows`. No separate file is created.

#### Diagram type selection

| Project type | Preferred diagram |
|---|---|
| Web app / API | `graph TD` — request flow top-down through layers |
| CLI tool | `flowchart TD` — args → parse → subcommands → output |
| Background worker / pipeline | `flowchart LR` — trigger → queue → stages → sinks |
| Monorepo | `flowchart LR` — one subgraph per package, shared infra shown once |
| Library | `flowchart TD` — public API surface → internal modules → external deps |
| Data-heavy (ETL, ML) | `flowchart LR` — sources → transforms → sinks |

Use multiple `subgraph` blocks to visually group related nodes. Name each subgraph after the
conceptual layer it represents (e.g. `subgraph Roles ["User Roles & Portals"]`,
`subgraph Services ["Business Service Layer"]`).

#### Depth requirements — include every layer that exists

1. **User roles / external actors** — all distinct user types (e.g. buyer, owner, admin), plus
   non-human callers (cron scheduler, webhook, upstream service). Use one node per distinct actor.

2. **Entry point** — the actual file and port that receives traffic (`main.py :8000`,
   `server.ts :3000`, `cmd/root.go`). Label with filename and bind address.

3. **Middleware** — auth/JWT verification, rate limiting, request validation, logging — as a
   named subgraph if a middleware layer is present in the code.

4. **Routers / handlers** — every distinct route group found in source, labeled with the actual
   path prefix and a short description (e.g. `"/api/campaign<br/>(Knapsack & Reach Optimization)"`).

5. **Services** — named service classes or modules containing business logic (e.g.
   `CampaignService`, `PaymentService`). One node per class/module.

6. **Repositories / data access** — if a distinct repository or DAO layer exists, show it between
   services and the DB entities. If services query the DB directly, skip this layer.

7. **Database entities** — individual tables or collections that are architecturally significant.
   Use cylinder shape `[("TableName")]`. Show schema relationships on edges with cardinality
   labels: `-->|"1:N"|`, `-->|"N:M"|`, `-->|"1:1"|`.

8. **External integrations** — third-party APIs and services by their real product name (Razorpay,
   Stripe, SendGrid, OpenAI, etc.). Use flag shape `>"Product Name"]`.

9. **Async / background layer** — Celery beat + worker, BullMQ, cron, etc. Show the broker
   (Redis, SQS) as a cylinder, the scheduler and worker as rectangles, and the schedule or
   trigger condition on the edge. Include the full polling/job loop if one exists.

10. **CI/CD** — if `.github/workflows` or equivalent is present, add a subgraph showing pipeline
    stages (lint → test → build → deploy).

Omit layers that genuinely don't exist. Never invent nodes.

#### Styling rules

Use `classDef` + `:::className` (not per-node `style` directives) for consistent color coding:

```
classDef buyer  fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#01579b;
classDef owner  fill:#efebe9,stroke:#5d4037,stroke-width:2px,color:#3e2723;
classDef admin  fill:#fbe9e7,stroke:#e64a19,stroke-width:2px,color:#bf360c;
classDef router fill:#ede7f6,stroke:#7e57c2,stroke-width:2px,color:#4a148c;
classDef service fill:#e8f5e9,stroke:#4caf50,stroke-width:2px,color:#1b5e20;
classDef db     fill:#fffde7,stroke:#fbc02d,stroke-width:2px,color:#f57f17;
classDef worker fill:#fff3e0,stroke:#ff9800,stroke-width:2px,color:#e65100;
classDef ext    fill:#eceff1,stroke:#607d8b,stroke-width:2px,color:#263238;
```

Adapt colors and class names to match the project's actual layer taxonomy. The key constraint is
that every distinct conceptual layer gets its own color class.

Node shapes by semantic role:
- `["label"]` rectangle → router, handler, file, module
- `("label")` rounded rectangle → service, use-case, orchestrator
- `{"label"}` diamond → decision / branch point
- `[("label")]` cylinder → database table, cache, message broker
- `>"label"]` flag/asymmetric → external third-party service or API

Node IDs must be descriptive camelCase: `authRouter`, `campaignSvc`, `userTable`, `redisCache`,
`razorpayGateway`. Never use single-letter IDs.

Node labels must use **real names from the codebase** — actual route paths, class names, table
names, product names. Use `<br/>` for multi-line labels (route path + short description).

Edge labels must describe the actual operation:
- `-->|"JWT verify"|` not just `-->`
- `-->|"SQL upsert"|`, `-->|"publishes event"|`, `-->|"Bearer token"|`
- `-->|"1:N"|` for schema relationships

For large diagrams, use `%%` section comments to group edge declarations:
```
%% Roles to Routers
%% Routers to Services
%% Services to DB
%% DB Relationships
%% Async Pipeline
```

Add a `%% Legend: color=layer` comment at the top of the diagram.

#### Target detail level

The diagram is the primary architectural reference in the README, not a summary. For a typical
API service, aim for: `graph TD` with `classDef` styling, one named subgraph per conceptual layer,
descriptive edge labels on every connection, DB cardinality on schema edges, and any async
pipelines shown end-to-end. Scale up or down based on actual codebase complexity — a small
project warrants a small diagram; a large multi-service project should have a correspondingly
detailed one.

---

### Phase 5 — Write the README

Produce a complete `README.md`. Sections appear in the order below.

#### Required sections

1. **Title + badges** — H1 project name; badges for CI status, test count, language version,
   key framework version — only if inferrable from workflows or manifests.

2. **Description** — 2–4 sentences: what it does, who uses it, why it exists. Follow with a
   **Features** bullet list if the project has 3 or more distinct capabilities worth calling out.

3. **System Architecture & Data Flows** — 1–2 sentence intro explaining what the diagram covers
   (which layers, which flows), then the full Mermaid diagram in a fenced ` ```mermaid ``` ` block.
   This is the complete, detailed diagram from Phase 4 — not a simplified version.

4. **Technology Stack** — language/runtime version, framework, key libraries. Bullet list or
   small table.

5. **Project Structure** — annotated directory tree. Skip `node_modules`, `__pycache__`, `.git`,
   `dist`, `build`. Annotate every meaningful directory and key files with a short comment.

6. **Environment Variables** — table: Variable | Required | Default | Description. Every variable
   from `.env.example` (or equivalent) must appear. Use `None` as the default when there is no
   fallback.

7. **Getting Started**
   - Prerequisites (what must already be installed, with versions)
   - Installation steps (numbered, copy-pasteable)
   - Running the dev server
   - Running background workers, if any (separate numbered steps)

8. **Containerization & Deployment** — if Dockerfile or compose files exist, show exact commands.
   If multiple compose files exist (dev vs prod), document both with their purpose.

9. **Testing** — how to run the suite. Include: full run command, single-file run command,
   coverage flag if configured. Note any environment quirks (e.g. in-memory DB shim).

10. **API Documentation** — if auto-generated docs are served (Swagger UI, ReDoc, GraphiQL),
    list their local URLs.

#### Conditional sections

- **API Reference** — only if routes are few enough to document inline (< ~15 endpoints)
- **Monorepo Packages** — if workspace config detected; one-line description per package
- **Contributing** — preserve from existing README if present; otherwise omit entirely
- **License** — preserve from existing README or infer from `LICENSE` file if present

#### Tone and style

- Developer-facing, professional, concise
- Second person throughout ("Run `npm install`", not "The user should run...")
- All commands in fenced code blocks with language tags (`bash`, `env`, `text`)
- LaTeX math (`$$...$$`) for any formulas present in the codebase (pricing models, algorithms)
- No marketing language — no "robust", "powerful", "seamless", "cutting-edge"

#### Placeholder policy

When information is genuinely unknown, write exactly:

```markdown
<!-- TODO: fill in -->
```

Never guess or invent content to fill a gap.

---

### Phase 6 — Output

Write the result to `<root>/README.md`, overwriting the existing file. Only this one file is
produced — no `ARCHITECTURE.md`, no separate diagram file.

Call `present_files` with the README path.

Print a brief change summary:
- Sections preserved verbatim from the original
- Sections rewritten
- Sections added (new)
- Diagram: which layers were detected and included
- Placeholders inserted and why

---

## Edge cases

**No README exists** → Create from scratch; use placeholders for anything not inferrable.

**Monorepo** → One README at root covering the whole repo. Diagram uses one subgraph per package
with shared infrastructure shown once. Link to per-package READMEs if they exist.

**Minimal codebase** (1–2 files) → Short README, no padding. Diagram can be 4–8 nodes — don't
force subgraphs or layers that don't exist.

**Private / internal tool** → Document as if the reader is a new team member on day one.

**Conflicting signals** (e.g. two start commands in different files) → Document both, add a
comment noting the discrepancy.

**Very large diagram (>60 nodes)** → Keep it in the single README diagram; use `%%` section
comments to keep the Mermaid source navigable. Do not create a separate file.

---

## Quality checklist (self-review before writing)

### README prose
- [ ] Every command is copy-pasteable and matches a real script, target, or entrypoint
- [ ] No version numbers or package names invented — all from manifest files
- [ ] Every env var listed exists in `.env.example` or is explicitly used in source
- [ ] No section omitted because it seemed unnecessary
- [ ] `<!-- TODO: fill in -->` used instead of guessing
- [ ] Existing accurate content preserved verbatim

### Mermaid diagram
- [ ] Every node is a real file, module, route, service, table, or product in the codebase
- [ ] No nodes invented — only what exists in the code
- [ ] `classDef` blocks defined and applied with `:::className` on every node
- [ ] Node shapes match semantic role (cylinder = DB/cache, flag = external API, rounded = service)
- [ ] Every edge has a label describing the real operation or relationship
- [ ] Real codebase names used throughout (route paths, class names, table names, product names)
- [ ] `%%` section comments group edge blocks in large diagrams
- [ ] Diagram embedded directly in README — no separate file created
- [ ] Bracket and quote balance verified before finalizing (no syntax errors)