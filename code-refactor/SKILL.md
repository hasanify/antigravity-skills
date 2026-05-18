---
name: code-refactor
description: Refactors messy, disorganized, or poorly structured codebases into production-grade, professional code — without altering any business logic, UI behavior, or functionality. Use this skill whenever the user asks to "refactor my code", "clean up this codebase", "restructure this project", "improve code quality", "make this production-ready", "organize my files", or shares code and says things like "this is a mess, can you fix it?", "turn this into a proper architecture", "make this maintainable", or "clean this up". Also trigger when the user shares a monolithic file and wants it split, asks to apply SOLID principles, or wants to remove code duplication. Always use this skill before attempting any large-scale code reorganization or architectural improvement.
---

# Code Refactor Skill

## Goal
Transform messy, disorganized, or poorly structured code into production-grade, professional code — **without altering any business logic, UI behavior, API contracts, or functionality**. The observable behavior of the system must be indistinguishable before and after refactoring.

---

## Phase 1: Analysis (Always Do This First)

Before writing a single line of code, perform a thorough analysis and output a clear plan.

### 1a. Identify Problem Areas
Scan the entire codebase and call out every instance of:
- **God objects / monolithic files** — files doing too many things
- **Mixed concerns** — data fetching alongside business logic or UI rendering
- **Code duplication** — repeated patterns that should be abstracted
- **Poor naming** — unclear variable/function/file names
- **Magic values** — hardcoded strings, numbers, URLs not in constants
- **Missing error handling** — silent failures, swallowed exceptions
- **AI comment pollution** — step comments, change-log notes, redundant narration
- **Dead code** — commented-out blocks, unreachable branches, unused imports
- **Improper async patterns** — missing awaits, unhandled rejections, inconsistent patterns
- **Missing types** — untyped parameters, `any` usage, missing interfaces

### 1b. Propose the New Structure
Output the proposed directory tree before touching any code. Adapt the folders to the actual stack — do not force irrelevant patterns. Common folders by context:

**Frontend (React/Vue/etc.)**
```
src/
├── components/        # Reusable UI components, one per file
├── hooks/             # Custom hooks (React)
├── services/          # Business logic, API orchestration
├── repositories/      # All data fetching / API calls
├── utils/             # Pure utility functions, formatters, validators
├── types/             # TypeScript interfaces and type definitions
├── constants/         # Named constants, enums, config values
└── pages/ or views/   # Route-level page components
```

**Backend (Node/Python/etc.)**
```
src/
├── controllers/       # Route handlers — thin, delegate to services
├── services/          # Business logic and orchestration
├── repositories/      # Database queries, one file per domain entity
├── models/            # ORM models, schemas, data structures
├── middleware/         # Request/response middleware
├── utils/             # Pure helpers, formatters, validators
├── constants/         # Named constants, enums
└── config/            # Environment and app configuration
```

Only include folders that have content. Explain the purpose of each.

### 1c. List Every File to Create or Modify
Enumerate all files, with one sentence on what each will own.

---

## Phase 2: Refactoring Rules

Apply ALL of the following, adapted to the language/framework in use.

### Architecture & Separation of Concerns
- One file per domain entity in `repositories/` — no cross-entity queries in one file
- Services orchestrate repositories and contain business logic — no DB queries in services
- Controllers/components are thin — they call services, they do not contain logic
- No data fetching inside UI components or controllers
- No business logic inside repositories

### Modularity
- Break every monolithic file into focused, single-responsibility modules
- Extract every reusable UI pattern into its own component
- Extract every repeated code block into a named utility function
- Keep functions under ~30 lines; if longer, decompose further

### Naming
- File names: `camelCase.ts`, `kebab-case.tsx`, or whatever the project convention is — be consistent
- Functions: verb phrases (`getUserById`, `formatCurrency`, `validateEmail`)
- Components: PascalCase nouns (`UserCard`, `OrderSummary`)
- Constants: `SCREAMING_SNAKE_CASE`
- Types/Interfaces: PascalCase, prefix interfaces with `I` only if the project already does this

### Constants & Config
- Replace every magic string/number with a named constant
- Move all environment-dependent values to config files or `.env`
- Group related constants together in domain-specific constant files

### Comments & Documentation
- Add JSDoc/docstrings to every exported function, class, hook, and module
- Module-level comment at the top of each file: its purpose and what it does NOT own
- Comments explain *why*, not *what* — the code explains what
- **Aggressively delete:**
  - Obvious comments (`// return the user`, `// call the API`, `// increment counter`)
  - Step comments (`// Step 1:`, `// Step 2:`)
  - Change-log comments (`// Added to fix X`, `// Updated by AI`)
  - Any comment a skilled engineer would find condescending or redundant

### Error Handling
- No silent failures — every `catch` block must log or rethrow
- Use consistent error types/classes across the codebase
- Async functions always have proper error boundaries
- Validate inputs at the boundary (controller/repository layer), not deep in business logic

### Typing
- TypeScript: full interfaces for all data shapes; no `any`; use generics where appropriate
- Python: full type hints on all function signatures
- Avoid implicit `any` from untyped third-party libraries — wrap with typed adapters

---

## Phase 3: Absolute Preservation Rules

**These are non-negotiable. Never alter:**

| Category | What to preserve |
|---|---|
| Business logic | Every conditional, calculation, validation rule, transformation, decision branch |
| Data models | All DB schemas, ORM models, Zod/Pydantic schemas — no field renames, no type changes |
| API contracts | All route paths, HTTP methods, request/response shapes, status codes |
| UI behavior | All layouts, CSS/Tailwind classes, animations, conditional rendering, interactions |
| External interfaces | All exported function/class/module signatures exposed to outside callers |

**If you are ever uncertain whether a change would affect observable behavior — do not make the change.**

---

## Phase 4: Output Format

For each file created or modified:

1. **File path** — full path (e.g., `src/repositories/userRepository.ts`)
2. **Responsibility** — 1–2 sentences on what this file owns and what it does not own
3. **Complete file contents** — the full refactored file, not a diff, not a snippet

After all files, provide a concise **Architectural Summary**:
- What changed and why
- How the project is now structured
- Key decisions made and the reasoning behind them

---

## Edge Cases & Judgment Calls

- **Incomplete codebases**: If only part of the codebase is shared, refactor what's given and note what else would need refactoring in a full codebase
- **Mixed stacks**: Adapt the folder structure to the actual stack — don't force a React structure onto a Django project
- **Legacy patterns**: If a legacy pattern (e.g., class components in React) is used consistently, preserve it — don't upgrade the pattern unless asked
- **Test files**: Preserve all test files and update imports to match new paths; do not change test logic
- **Large codebases**: If the codebase is very large, ask the user which modules to tackle first, or proceed in prioritized batches

---

## Reference: Folder Decision Guide

| Has this? | Put it in... |
|---|---|
| API/DB calls | `repositories/` |
| Business rules, calculations, orchestration | `services/` |
| Reusable React/Vue components | `components/` |
| React hooks (`use*`) | `hooks/` |
| Pure functions, formatters, validators | `utils/` |
| TypeScript interfaces, types | `types/` |
| Enums, string constants, magic values | `constants/` |
| Express/FastAPI route handlers | `controllers/` |
| Auth, logging, rate-limiting middleware | `middleware/` |
| DB connection, env vars, app config | `config/` |