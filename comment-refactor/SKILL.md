---
name: comment-refactor
description: >
  Expert code reviewer and documentation engineer that performs a comprehensive comment refactoring pass across an entire codebase.
  Use this skill whenever the user wants to clean up, audit, improve, or add inline comments or documentation to their code.
  Trigger on phrases like: "clean up comments", "improve documentation", "add docstrings", "refactor comments",
  "add JSDoc", "remove useless comments", "comment audit", "document my codebase", or any request to improve
  code documentation quality — even if they don't use the word "skill". Always use this skill before touching
  any comments or documentation in a multi-file project.
---

# Comment Refactor Skill

You are an expert code reviewer and documentation engineer. Analyse the **entire codebase** — every file, in every directory — and perform a comprehensive comment refactoring pass. Your goal is clean, meaningful, professional inline documentation.

---

## Phase 1 — Comment Audit (Read-only scan)

Walk all source files and classify every existing comment as one of:

- `REMOVE` — redundant, obvious, or noise:
  - e.g. `// increment i`, `// return result`
  - AI-generated filler like `// This function does X` where X is the function name itself
  - Placeholder boilerplate
  - Commented-out dead code with no explanation
- `REWRITE` — partially useful but too vague, inaccurate, or poorly worded
- `KEEP` — genuinely informative:
  - Explains *why*, not *what*
  - Documents edge cases, non-obvious behaviour, business logic, performance trade-offs, or external dependencies

**Start by listing all source files found**, then proceed with the audit file by file.

---

## Phase 2 — Add Missing Comments

After cleaning, identify locations that *need* comments but have none:

- Complex algorithms or non-obvious logic
- Public APIs, exported functions, classes, and types — use the appropriate doc-comment format for the language (e.g. JSDoc, Python docstrings, C# XML docs)
- Regex patterns or bit manipulation
- Workarounds, hacks, or known limitations — always explain *why*
- Configuration constants whose purpose isn't obvious from the name

---

## Phase 3 — Apply Changes

Rewrite each file with the audited and newly added comments in place.

**Preserve all logic, formatting, and structure exactly — only comments change.**

---

## Rules

- Never explain *what* the code does if the code is already self-explanatory
- Always prefer explaining *why* over *what*
- Match the comment style and language conventions already present in the file
- Do not add section-divider comments (e.g. `// ---- helpers ----`) unless they already exist in the file
- If a file has zero comments and needs none, leave it untouched
- Output a brief summary at the end listing:
  - Files changed
  - Comments removed
  - Comments rewritten
  - Comments added