---
name: smart-commit
description: >
  Analyses all changed files in a Git repository and automatically stages and commits them in
  logical, atomic phases — each with a meaningful Conventional Commits-formatted message.
  Use this skill whenever the user wants to commit changes, asks to "smart commit", "auto commit",
  "commit my changes", "group my commits", or says anything like "commit these files intelligently"
  or "create meaningful commits from my changes". Always trigger this skill for any multi-file
  commit workflow where the user wants well-organised, atomic git commits. Do NOT use for
  single-command git operations the user spells out explicitly.
---

# smart-commit

Analyse all repository changes, group them into logical atomic commits, generate Conventional
Commit messages, confirm with the user, then execute in order.

---

## Phase 1 — Change Discovery

Run all three discovery commands:

```bash
git diff --cached --name-status          # staged
git diff --name-status                   # unstaged tracked
git ls-files --others --exclude-standard # untracked
```

If all three return empty output → print `Nothing to commit. Working tree clean.` and stop.

For each changed file, retrieve its diff:
- Tracked file (staged or unstaged): `git diff HEAD -- <file>`
- Untracked file: read the file directly
- If a file appears in both staged and unstaged lists, combine both diffs

Extract per file:
| Field | What to determine |
|---|---|
| path | Full relative path |
| extension | File extension (infer language/type) |
| change_type | `added` / `modified` / `deleted` / `renamed` |
| nature | `feat` / `fix` / `refactor` / `style` / `test` / `docs` / `chore` / `perf` / `ci` / `build` |
| coupled_to | Other files sharing imports, same feature area, or same module |

---

## Phase 2 — Logical Grouping

Group files into **atomic commit units** using these rules (in priority order):

1. **Feature cohesion** — files that implement the same feature or user-facing change go together
2. **Layer separation** — separate concerns unless tightly coupled (e.g. don't mix DB schema + UI unless inseparable)
3. **Change type** — refactors separate from features; config/tooling separate from business logic
4. **Dependency order** — if commit A is required for commit B to make sense, A comes first

Rules:
- Each file belongs to exactly one group
- Each group = one planned commit
- Deleted files must use `git rm`; renamed files handled as a remove+add pair

---

## Phase 3 — Commit Message Generation

For each group, generate a message following **Conventional Commits**:

```
<type>(<scope>): <short imperative summary — max 72 chars>

<body — optional, only when change needs explanation>
- What changed and why (not how)
- One bullet per key point
- Max 5 bullets

<footer — only if applicable>
BREAKING CHANGE: <description>
Closes #<issue>
```

**Valid types:** `feat`, `fix`, `refactor`, `style`, `test`, `docs`, `chore`, `perf`, `ci`, `build`

**Scope:** affected module, route, component, or layer — e.g. `auth`, `api`, `dashboard`, `db`, `config`

---

## Phase 4 — Preview & Confirmation

Display the full plan before touching anything:

```
Planned commits (N):

[1] feat(auth): add JWT refresh token rotation
    Files: src/auth/refresh.ts, src/auth/middleware.ts, tests/auth.test.ts

[2] fix(api): correct null handling in user profile endpoint
    Files: src/routes/user.ts

[3] chore(config): update ESLint rules for import ordering
    Files: .eslintrc.js
```

Then ask:

```
Proceed with all commits? [y] Yes / [e] Edit plan / [n] Cancel
```

Handle responses:
- **`y`** → proceed to Phase 5
- **`e`** → let the user rename messages, move files between groups, or split/merge groups, then re-display the plan and ask again
- **`n`** → print `Aborted. No changes made.` and stop

---

## Phase 5 — Execution

For each planned commit in order:

```bash
# For regular files:
git add <file1> <file2> ...

# For deleted files:
git rm <file>

# Commit:
git commit -m "<subject>" -m "<body>"   # use -m twice if body is non-empty
```

- Report each commit as it succeeds: `✓ [1/3] feat(auth): add JWT refresh token rotation`
- If a `git commit` fails, surface the exact Git error and stop (do not continue to next commits)

After all commits, print a summary:

```
✓ 3 commits created
  a1b2c3 feat(auth): add JWT refresh token rotation
  d4e5f6 fix(api): correct null handling in user profile endpoint
  g7h8i9 chore(config): update ESLint rules for import ordering
```

Get the short hashes with: `git log --oneline -<N>`

---

## Edge Cases & Constraints

| Situation | Behaviour |
|---|---|
| No changes at all | Print "Nothing to commit." and exit immediately |
| File in both staged + unstaged | Combine full diffs for analysis; `git add` will pick up everything |
| Deleted file | Stage with `git rm`, not `git add` |
| Renamed file | Treat as a remove+add pair in the same group |
| Only one logical group found | Still show the plan and ask for confirmation |
| `.gitignore`d files appear in untracked list | Skip — `git ls-files --others --exclude-standard` already filters them |
| Commit fails mid-run | Stop, report the error, do not attempt remaining commits |
| **Never** | Push, amend existing commits, or stage files not in the plan |