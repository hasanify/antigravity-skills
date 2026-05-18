---
name: ui-redesign-orchestrator
description: >
  Orchestrates a structured UI/UX redesign workflow. This skill is manually
  invoked only — trigger it exclusively when the user explicitly calls
  /ui-redesign-orchestrator. Do NOT trigger it automatically based on design
  intent, UI mentions, or redesign-related phrasing. Once invoked, run Phase 1
  (constraint gathering) then Phase 2 (direction proposals) in order.
---

# UI Redesign Orchestrator

A two-phase skill for structured, high-quality UI redesign proposals. Always
run both phases in order. Never generate design proposals before completing
Phase 1.

---

## PHASE 1 — Gather Design Constraints

Before any design work, elicit four constraints from the user using the
`ask_user_input_v0` tool. Ask all four at once in a single tool call — never
split them across multiple messages.

Present a short conversational opener first, e.g.:

> "Before I propose directions, I need four quick answers to make sure the
> proposals actually fit your project."

Then call `ask_user_input_v0` with these four questions:

```
Q1 — Component library
"Should the redesign use shadcn/ui components, or build custom components from scratch?"
Options: ["Use shadcn/ui", "Custom components from scratch"]

Q2 — Colour palette
"Should the redesign reuse the existing colour palette, or define a completely new one?"
Options: ["Reuse existing palette", "Define a new palette"]

Q3 — Design inspiration
"How much should the existing UI influence the redesign?"
Options: ["Blank slate — ignore existing UI entirely", "Loose reference — take structural cues but rethink everything", "Refinement — keep the bones, improve the skin"]

Q4 — UI direction & style (free text OR single select)
"What feeling or aesthetic should the new UI evoke?"
Options: ["Corporate & trustworthy", "Playful & bold", "Dark & techy", "Warm & human", "Editorial & content-first", "Luxury & refined", "Surprise me"]
```

**Wait for the user's answers before proceeding. Do not generate proposals yet.**

---

## PHASE 2 — Direction Proposals

Once all four answers are collected, read the `frontend-design` skill:

> `/mnt/skills/public/frontend-design/SKILL.md`

Apply it to generate **3–5 distinct UI direction proposals**. Follow all
constraints from Phase 1 answers (see constraint rules below).

### Proposal format

Each proposal must include all of these sections:

**Direction name** — A short, evocative label (e.g. "Neomorphic Calm",
"Brutalist Grid", "Minimal Tech", "Warm Editorial", "Dark Glassmorphism").
Avoid generic names like "Modern Clean" or "Flat Design".

**Visual description** — 2–3 sentences covering: typography style, colour
mood, layout personality, and motion character.

**Key design choices** — Bullet list of 4–6 specific, concrete decisions:
  - Font pairing (name actual typefaces, not categories)
  - Background treatment (e.g. "textured linen at 4% opacity over #FAF7F2")
  - Component style (e.g. "pill-shaped buttons with 2px inset shadow")
  - Accent use (e.g. "single accent: terracotta #C0533A, used only on CTAs")
  - Spacing philosophy (e.g. "generous whitespace, 8pt grid, wide margins")
  - Animation approach (e.g. "spring physics on hover, 200ms ease-out reveals")

**Best suited for** — One sentence on the product type or audience this
direction fits best.

**shadcn compatibility note** — *(include only if user chose shadcn/ui)*
  - Rate fit: High / Medium / Low
  - Specify exactly which CSS variables (`--radius`, `--background`, `--foreground`,
    `--primary`, `--muted`, etc.) must be overridden to match this direction
  - Note any components that need `className` overrides or custom `cva` variants
    beyond what CSS variables alone can achieve (e.g. custom border treatments,
    non-standard shadow styles, motion additions)
  - **shadcn components must never look out-of-the-box.** Every component used
    in implementation must be visually adapted to the chosen direction through
    CSS variable overrides, Tailwind utility overrides, or wrapper styling.
    Default shadcn grey-on-white aesthetics are not acceptable output.

### Distinctness rules

Proposals must be **genuinely distinct**. No two proposals may share:
- The same typography style (e.g. two serif proposals are not allowed)
- The same colour temperature (e.g. two warm proposals are not allowed)
- The same layout personality (e.g. two grid-heavy proposals are not allowed)

### Anti-defaults

Avoid these lazy AI defaults across all proposals:
- ❌ Purple or violet as a primary accent
- ❌ Inter as the sole or primary typeface
- ❌ Card-grid-on-white as the default layout
- ❌ Vague descriptors like "clean", "modern", "sleek" without specifics
- ❌ Gradient hero sections unless they serve the stated direction
- ❌ shadcn components left at their default visual appearance (grey borders, default radius, stock colours)

---

## Constraint rules (carry into every proposal)

| Phase 1 answer | Constraint applied |
|---|---|
| shadcn/ui | All proposals must be shadcn-compatible; include shadcn note per proposal |
| Reuse existing palette | Proposals adapt direction to the existing colours; do not replace them |
| Blank slate | Proposals must not reference or echo any patterns from the existing UI |
| Style direction given | At least 2 of the proposals must directly interpret that direction; others may reinterpret or interestingly contrast it |
| "Surprise me" | Full creative freedom — range widely across temperature, era, and aesthetic register |

---

## Closing the proposals

After presenting all proposals, close with:

> "Which direction resonates — or would you like me to blend elements from
> multiple proposals before we move to implementation?"

If the user picks one (or a blend), proceed to implementation using the
`frontend-design` skill to produce the actual code.

---

## Notes for implementation phase

Once a direction is chosen:
- Apply the `frontend-design` skill fully for the implementation
- Honour all Phase 1 constraints throughout the code
- Use the exact typefaces, colours, and spacing values from the chosen
  proposal — do not drift toward defaults at implementation time
- If shadcn was chosen, scaffold with shadcn primitives and apply CSS
  variable overrides as noted in the chosen proposal's shadcn note
- **shadcn components must be visually customised** — always override the relevant CSS variables and apply Tailwind utility classes so components match the chosen direction. Shipping a shadcn `<Button>` or `<Card>` that looks indistinguishable from the library default is not acceptable
- **Fully responsive** — every layout must work fluidly across mobile, tablet, and desktop. Use responsive Tailwind breakpoints (`sm:`, `md:`, `lg:`) throughout. No fixed widths that break on small screens, no horizontal overflow, no font sizes or spacing that only make sense at one viewport. Test mentally at 375px, 768px, and 1280px before considering the implementation complete
- **Never alter text copy** — preserve all original labels, headings, button text, descriptions, and content exactly as-is. The redesign changes only visual presentation — colours, typography style, spacing, layout, component styling — never the words themselves. Do not rewrite, rephrase, shorten, or "improve" any copy from the existing UI
- **No placeholders or dummy content** — never output `[Component here]`,
  `// TODO`, lorem ipsum, `<Placeholder />`, empty stub components, or any
  stand-in that defers real implementation. Every component, section, and
  element in the output must be fully implemented and functional. If something
  is out of scope, omit it entirely rather than scaffolding a shell for it.