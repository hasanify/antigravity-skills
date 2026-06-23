---
name: seo-geo-optimizer
description: >
  Audits and rewrites user-facing web copy to improve both traditional SEO and GEO
  (Generative Engine Optimisation) — the signals that determine whether LLMs cite,
  quote, or surface a page. Works inside Next.js, React, or any frontend project.
  Use this skill whenever the user wants to audit site copy, check SEO health, assess
  GEO citability, rewrite headings or meta tags, sharpen CTAs, find weak or duplicate
  content, improve ranking, or make content more visible in AI search responses. Trigger
  for casual requests like "audit my homepage", "rewrite my hero text for SEO", "why
  isn't my product showing up in AI answers", "make this CTA stronger", "check my meta
  descriptions", or "improve my landing page copy". Run audit-only, rewrite-only, or
  audit-then-rewrite depending on what the user asks. Always prefer this skill over
  ad-hoc copy analysis or edits when the goal is structured SEO/GEO output.
---

# SEO + GEO Copy Optimizer

Audit and rewrite user-facing copy so it performs better in both traditional search
engines and generative AI responses, without changing the original voice, meaning, or
factual claims.

The skill runs in three modes. Choose based on what the user asks:

- **Audit only** — score copy, flag issues, produce a priority action list
- **Rewrite only** — take copy blocks and produce optimised versions with rationale
- **Audit + Rewrite** — full pipeline: scan, score, then rewrite flagged blocks in the
  same report

If the user's intent is ambiguous, default to Audit + Rewrite.

---

## Input Modes

Accept any of these. Ask the user which applies if unclear.

**File path or glob** — `src/app/**/*.tsx`, `content/*.mdx`, `public/locales/en/*.json`

**Pasted copy block** — raw text treated as a single component

**URL** — fetch the page, strip nav/footer boilerplate, extract visible copy; note that
alt text and JS-rendered content may be incomplete

**Audit report** — if the user already has an audit and wants rewrites only, accept the
report as input and rewrite the flagged blocks in priority order

---

## Step 1 — Establish Context

Before doing anything else, confirm or infer:

1. **Product identity**: name, core category, primary differentiator
2. **Target audience**: who the primary user is
3. **Tone register**: auto-detect from the copy, or ask. Classify as formal, casual,
   technical, or friendly.
4. **Hard constraints**: meta title 50–60 chars, meta description 120–158 chars, any copy
   the user has marked as legally fixed or factually locked

If product context is missing, ask one question: "What does this product do, and what
category does it compete in?" Do not audit or rewrite without this.

---

## Step 2 — Define the Two Keyword Layers

Derive these from the product context and the copy itself. Every audit score and every
rewrite decision depends on them.

**Layer 1 — Product layer**
The product's name, branded feature names, proprietary terminology, and unique claims.
For a sprint planning tool called Planr: "Planr", "Smart Sprints", "AI task routing".

**Layer 2 — Category layer**
The broader market the product belongs to: "project management software", "agile planning
tool", "team collaboration app".

Both layers must appear in optimised copy. A page strong on Layer 1 but weak on Layer 2
fails GEO — LLMs cannot classify the product. A page strong on Layer 2 but weak on Layer
1 fails brand differentiation.

Document both lists before scoring or rewriting.

---

## Step 3 — Locate and Classify Copy (Audit + Rewrite mode / Audit mode)

Skip this step if the user provides a pasted block or an existing audit report.

Scan files or the fetched URL for these copy types:

| Type | Where to Look |
|------|---------------|
| Page title (`metadata.title`, `<title>`) | `layout.tsx`, `page.tsx`, `head.tsx` |
| Meta description | Same as above |
| H1 | Page components, MDX |
| H2–H4 | Page and section components |
| Hero headline + subheadline | Hero/Banner components |
| CTA text (buttons, action links) | Throughout |
| Feature descriptions | Feature/Card components |
| Alt text | All JSX/TSX `<img>` and `next/image` |
| Footer taglines | Footer component |
| i18n strings | `locales/*.json`, `messages/*.json` |

For each item, record: file path + line number, copy type, raw text, page/route.

---

## Step 4 — Score Each Copy Block (Audit mode)

Score every block on four dimensions, each out of 10. Average applicable dimensions for
a composite score. Flag anything below 5 as high priority.

**Keyword Alignment (SEO)**
Are the right keywords present, at natural priority positions?

- 9–10: Both layers present, primary keyword early in the copy
- 6–8: One layer strong, the other weak or absent
- 3–5: Generic category language only, no product specificity
- 0–2: Neither layer; this copy could belong to anything

**GEO Entity Clarity**
Can an LLM understand what the product is, does, and who it serves from this block alone?

- 9–10: Product name + category + use case + differentiator all present
- 6–8: Name or category present, but context is needed to complete the picture
- 3–5: Vague; an LLM would classify this as generic marketing copy
- 0–2: No entity signal at all

A hero line like "Work smarter, not harder" gives an LLM nothing to cite. "Planr cuts
sprint planning time by 40% for remote engineering teams" is citable.

**Specificity**
Does the copy make concrete, verifiable claims?

- 9–10: Specific numbers, named outcomes, named user types, or concrete comparisons
- 6–8: Some specificity, some vague qualifiers
- 3–5: Mostly generic — "powerful", "easy to use", "the best"
- 0–2: Pure superlatives or empty marketing language

**CTA Strength** (CTAs only — mark N/A for all other copy types)
Does the CTA state a clear action and signal a benefit?

- 9–10: Action verb + specific object + implicit or explicit benefit
- 6–8: Clear action, benefit absent or vague
- 3–5: Ambiguous or passive
- 0–2: No CTA, or a CTA that works against the user journey

---

## Step 5 — Flag Structural Issues (Audit mode)

Check across all pages/components for:

- Duplicate or near-identical blocks across different routes
- Pages with no `metadata.description`
- Empty or missing alt text on non-decorative images
- Multiple H1s per page, or zero H1s
- Heading hierarchy breaks (H3 before H2, H4 with no parent H3)
- Feature descriptions under 20 words
- Pages with no category-layer keywords (pure brand copy)
- Pages with no product-layer keywords (pure generic copy)

---

## Step 6 — Rewrite Copy Blocks (Rewrite mode)

Apply these techniques to every block being rewritten.

**Dual-layer keywords**
Place at least one Layer 1 term in the first sentence of H1s, hero headlines, and meta
descriptions. Include at least one clear Layer 2 phrase per page, not necessarily per
block. When both layers must coexist in short copy, spread them across the headline and
subheadline rather than forcing both into one sentence.

**GEO techniques**

*Entity disambiguation* — define the product category at first mention. "Planr, a sprint
planning tool for remote engineering teams" is more citable than "Planr" alone.

*Answer-friendly phrasing* — structure claims as direct answers. "Planr syncs with Jira
in real time" is more likely to surface in an AI response than "deep integrations with
your existing stack".

*Definition-first structure* — lead with what the product is before what it does. LLMs
build entity graphs from category context first.

*Specific, verifiable claims* — "40% faster sprint planning" is citable. "Faster than
ever" is not. If no data exists, ask the user rather than inventing a number.

*FAQ reformatting* — where feature descriptions are vague, suggest an FAQ-style
alternative. Mark these as optional restructures, not enforced rewrites.

*Structured data hints* — flag where schema markup (`Product`, `FAQPage`,
`SoftwareApplication`) would amplify GEO signal. These are annotations only, not code.

**SEO fundamentals**

Primary keyword in the first 100 characters of H1, meta title, and meta description.
Secondary keywords in H2–H3 headings and the opening sentence of feature descriptions.
Use synonyms and related terms rather than exact-match repetition — "sprint planning
software" and "agile planning tool" reinforce each other; three uses of the same phrase
do not.

If heading hierarchy is out of order, note this and rewrite to reflect the correct order.
Do not restructure the page; flag it for the developer.

Meta title: 50–60 characters. Meta description: 120–158 characters. Cut to fit while
keeping the primary keyword intact.

**CTA formula**
Every CTA rewrite needs three components: an action verb (Start, See, Get, Try, Book,
Download) + a specific object (your free trial, the live demo, your first sprint) + an
implicit or explicit benefit (save 5 hours, no credit card required).

Never invent the benefit. If the product offers a free trial, use it. If the user hasn't
confirmed the offer, ask first.

Weak-to-strong examples:
- "Learn more" → "See how remote teams cut planning time in half"
- "Get started" → "Start your 14-day free trial — no credit card"
- "Click here" → "Download the 2024 sprint planning guide"
- "Contact us" → "Talk to the team — we reply within one business day"

**Rewrite constraints — non-negotiable**

These override all optimisation goals:

1. No factual changes. "99.9% uptime" stays "99.9% uptime".
2. No new promises. Do not add claims the original copy does not make.
3. No tone drift. A casual brand stays casual. Formal stays formal.
4. No legal alteration. Disclaimers, warranty language, regulatory statements — mark
   locked and leave them unchanged.
5. No invented data. Ask the user for any statistic a rewrite would need.
6. Character limits bind. A meta description at 200 characters has failed.

---

## Step 7 — Write-Like-a-Human Rules

Apply to all written output: audit notes, scores, rationale, rewritten copy, summaries.

**Banned words and phrases — never use:**
delve into, underscore, pivotal, realm, harness, illuminate, shed light on, bolster,
streamline, facilitate, foster, leverage, cutting-edge, state-of-the-art, innovative,
revolutionary, game-changing, transformative, groundbreaking, seamless, robust, scalable,
holistic, synergy, paradigm shift, nuanced, comprehensive, best practices, Certainly,
Absolutely, Of course, Great question, I'd be happy to, That being said, At its core,
To put it simply, A key takeaway is, It's worth noting, It's important to note,
In today's world, In the realm of, When it comes to, At the end of the day,
generally speaking, broadly speaking, various, a number of, a wide range of, many people,
truly, literally, essentially, ultimately, clearly, obviously, undoubtedly, fascinating,
incredible, wonderful, amazing, utilize (→ use), commence (→ start).

**Structural rules:**
Write in prose unless items are genuinely parallel. Do not bold random phrases for
emphasis. Never open a note with "It is important to note." Start immediately — no
preamble about what you are about to do.

**Sentence rhythm:**
Vary sentence length. Short sentences land harder. Longer ones carry context. Never write
three consecutive sentences beginning with the same word.

**Specificity:**
Name the specific problem. "The H1 lacks a category keyword" beats "the heading could be
stronger." If a claim cannot be made specifically, cut it.

**AI sentence patterns to avoid in all output:**
- "Not a weakness, but an opportunity"
- "This is not just X, it's Y"
- "[Product] allows you to unlock..."
- "Whether you are a beginner or expert..."
- "In an era where users expect..."
- "As [industry] continues to evolve..."

**Self-check before finalising any rewrite:**
Could this sentence appear in a template for a different product? If yes, rewrite it.

---

## Step 8 — Produce the Report

Output a single Markdown report. Combine audit and rewrite sections when both modes run.

```
# SEO + GEO Copy Report
**Project:** [name or URL]
**Date:** [today]
**Mode:** [Audit / Rewrite / Audit + Rewrite]
**Tone register:** [formal / casual / technical / friendly]
**Product layer terms:** [list]
**Category layer terms:** [list]

---

## Executive Summary

[3–5 sentences: overall health, most critical issues, quick wins. Audit-only or
Audit+Rewrite mode only.]

---

## Page / Component Results

### [Page name or route — e.g. `/` Homepage]

#### [Copy type — e.g. H1]

**Original:**
> [exact text]

**Scores:** *(audit mode)*
| Dimension | Score | Note |
|-----------|-------|------|
| Keyword Alignment | X/10 | |
| GEO Entity Clarity | X/10 | |
| Specificity | X/10 | |
| CTA Strength | X/10 or N/A | |
| **Composite** | **X/10** | |

**Issue:** [specific problem — what is weak and why] *(audit mode)*
**Priority:** High / Medium / Low *(audit mode)*

**Rewritten:** *(rewrite mode)*
> [new text]

**Rationale:** [2–4 sentences: which keyword was added, which layer it serves, what the
rewrite fixes. No generic statements.] *(rewrite mode)*

**Character count:** [original → rewritten] *(meta titles and descriptions only)*
**Constraint flags:** [e.g. "legal language in sentence 2 preserved unchanged"]

---

[Repeat for each block]

---

## Structural Issues *(audit mode)*

[Each flag with file/location]

---

## Structured Data Hints *(rewrite mode)*

[Pages/components where schema markup would help, with schema type and field to populate]

---

## What Was Not Changed *(rewrite mode)*

[Copy blocks left unchanged, with reason: out of scope, legally protected, already
scoring 9+, or user instruction]

---

## Priority Action List *(audit mode)*

1. [Highest-impact fix]
2. ...

---

## Score Summary *(audit mode)*

| Page/Component | Keyword | GEO | Specificity | CTA | Overall |
|----------------|---------|-----|-------------|-----|---------|
| Homepage `/` | X | X | X | X | X |
```

### Optional JSON Output

For CI pipeline integration, produce this alongside the Markdown report when requested:

```json
{
  "date": "YYYY-MM-DD",
  "project": "...",
  "mode": "audit | rewrite | audit+rewrite",
  "tone_register": "...",
  "product_layer_terms": [],
  "category_layer_terms": [],
  "pages": [
    {
      "route": "/",
      "name": "Homepage",
      "blocks": [
        {
          "type": "h1",
          "file": "src/app/page.tsx",
          "line": 14,
          "original": "...",
          "rewritten": "...",
          "scores": {
            "keyword_alignment": 6,
            "geo_entity_clarity": 4,
            "specificity": 5,
            "cta_strength": null
          },
          "composite": 5.0,
          "priority": "high",
          "issue": "...",
          "rationale": "...",
          "constraint_flags": [],
          "layers_addressed": ["product", "category"]
        }
      ]
    }
  ],
  "structural_issues": [],
  "structured_data_hints": [],
  "unchanged_blocks": [],
  "priority_actions": []
}
```

---

## Edge Cases

**No product context**: Ask before starting. One question is enough — "What does this
product do, and what category does it compete in?"

**Large projects (50+ components)**: Ask the user to prioritise routes. Start with the
homepage, primary landing pages, and highest-traffic product pages.

**i18n files**: Audit the default locale only (usually `en`). Note other locales exist
but are out of scope unless the user asks.

**Minified or build output** (`/dist`, `/.next`): Redirect to source files.

**Purely decorative copy** (nav labels, legal boilerplate): Score N/A on GEO dimensions.
Flag as out of scope.

**URL + JS-heavy pages**: Note that dynamically rendered content may be missing. Recommend
file-path input for full coverage.

**Copy already scoring 8+**: Do not rewrite for the sake of it. Note the score and
suggest minor optional refinements only.

**Very short blocks** (under 8 words, e.g. navigation labels): Out of scope unless the
user specifically asks.

**Multilingual projects**: Default locale only. Note that keyword strategy varies by
language and market — other locales need separate review.

**Brand name format**: If the user specifies exact capitalisation or spacing for the
product name, preserve it verbatim.

**Conflicting audit priority and user instruction**: If the audit flagged a block as high
priority but the user says leave it alone, respect the user. Document the conflict in the
report.

**Rewrite-only with no surrounding context**: Rewrite the block, but note that keyword
alignment cannot be fully verified without seeing the page H1 and body copy.
