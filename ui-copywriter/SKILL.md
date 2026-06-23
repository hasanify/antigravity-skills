---
name: ui-copywriter
description: >
  Rewrites UI strings — labels, tooltips, error messages, empty states, button copy, onboarding
  text, confirmations, and any other interface strings — so they feel natural and human, not
  corporate or procedural. Use this skill whenever an AI coding agent (Cursor, Claude Code,
  Antigravity, or similar) needs to audit or improve UI copy in a codebase. Trigger on phrases
  like "rewrite UI copy", "improve interface text", "make copy more human", "audit UI strings",
  "fix error messages", or whenever the agent is tasked with touching user-facing text across
  files. Also trigger for single-string rewrites requested inline during a coding session.
---

# UI Copywriter

Rewrites user-facing interface strings to feel warm, plain, and direct — as if written by a
thoughtful person, not a product team trying to sound smart.

---

## Core principles

- **Plain language.** Use words a non-technical person would reach for naturally.
- **Short and scannable.** Prefer one idea per sentence.
- **Emotionally appropriate.** Match the tone to the moment: errors should feel helpful, not
  alarming; confirmations should feel reassuring, not bureaucratic; empty states should feel
  inviting, not broken.
- **Active voice.** "We couldn't find your file" beats "Your file could not be located."
- **Precise where it matters.** Keep technical terms (API key, OAuth, webhook) when the
  audience genuinely needs them. Don't force plain language where precision matters more.

---

## What to avoid

| Pattern | Example | Fix |
|---|---|---|
| Nominalisation | "Make a selection" | "Choose" |
| Passive voice | "Your request has been submitted" | "We've got your request" |
| Corporate filler | "Please be advised that…" | Just say the thing |
| AI opener filler | "Great! Sure! Of course!" | Cut entirely |
| Em dashes for style | "We'll save it — automatically" | "We save it automatically" |
| Over-explanation | "This action cannot be undone. Once deleted, your data will be permanently removed and unrecoverable." | "This can't be undone." |
| Alarm in errors | "Error: Operation failed." | "Something went wrong — try again." |
| Vague empty states | "No data." | "Nothing here yet. Add your first X to get started." |
| Jargon on general UI | "Initialise the authentication flow" | "Sign in" |

---

## Output format

Always return a Markdown table with three columns:

```
| Original | Rewritten | Notes |
|---|---|---|
```

**Notes column rules:**
- Use sparingly — only when the rewrite made a meaningful assumption, or the original's intent
  was genuinely ambiguous.
- Leave blank otherwise. Do not explain every rewrite.

---

## Handling bulk input from a codebase

When an agent passes multiple strings at once (e.g. extracted from i18n files, JSX, or
constants files):

1. Process all strings in a single table.
2. Preserve the original string exactly in the Original column (no trimming or escaping changes).
3. If a string is already good — plain, warm, direct — mark it with `✓ Keep` in the Rewritten
   column and leave Notes blank.
4. Do not reorder rows. The agent may use row position to map rewrites back to source locations.
5. If a string contains a placeholder (e.g. `{name}`, `%s`, `{{count}}`), preserve the
   placeholder format exactly.
6. If a string is clearly for internal/developer use (e.g. log messages, console errors, debug
   labels), flag it in Notes as "Internal — skip?" and leave Rewritten blank.

---

## Tone by context

Use the string's label, key name, or surrounding code context to infer the emotional moment,
then match it:

| Moment | Tone | Example rewrite |
|---|---|---|
| Error / failure | Calm, helpful, actionable | "We couldn't save that. Check your connection and try again." |
| Confirmation / success | Warm, brief, reassuring | "All saved." |
| Destructive action | Direct, not alarming | "Delete this? It can't be undone." |
| Empty state | Inviting, not broken | "No projects yet. Create one to get started." |
| Onboarding | Friendly, low-pressure | "Let's set up your account. It takes about two minutes." |
| Loading | Calm, active | "Loading your dashboard…" |
| Permissions / consent | Clear, no pressure | "We'd like to send you product updates. You can turn this off any time." |
| Button / CTA | Short verb phrase | "Save changes", "Get started", "Try again" |
| Tooltip | One sentence, no period needed for very short tips | "Drag to reorder" |

---

## Quick reference: common rewrites

| Original pattern | Rewritten pattern |
|---|---|
| "An error occurred" | "Something went wrong" |
| "Operation successful" | "Done!" / "All saved." |
| "Please enter a valid email address" | "That doesn't look like an email address" |
| "Are you sure you want to delete?" | "Delete this? It can't be undone." |
| "No results found" | "No results — try a different search" |
| "Your session has expired" | "You've been signed out. Sign back in to continue." |
| "Access denied" | "You don't have permission to view this" |
| "Loading, please wait" | "Loading…" |
| "Click here to continue" | "Continue" |
| "Submit" (generic form) | Use the action: "Save", "Send", "Book", "Sign up" |