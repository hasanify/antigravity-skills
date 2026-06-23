---
name: website-seo-audit-crawler
description: >
  Give the agent a website URL. The agent installs Playwright, crawls every internal page
  via headless browser, extracts all copy and metadata, runs a full SEO + GEO audit on
  every page, then outputs two files: (1) audit_report.pdf — a polished client-facing
  proposal/pitch with findings and issues only, no fixes — and (2) improvements.md — a
  complete, actionable fix list the agent can apply if the client is won. Trigger whenever
  the user provides a URL and asks for a site audit, SEO audit, website review, or wants
  to generate a client proposal from a website. Do NOT trigger for single-file or
  paste-based copy audits — use seo-geo-optimizer for those.
---

# Website SEO + GEO Audit — Full Crawler

Crawl a live website, audit every page for SEO and GEO health, and produce two deliverables:

- **`audit_report.pdf`** — a professional pitch/proposal for the website owner. Findings and
  scores only. No rewrites, no fix instructions. Designed to land a client.
- **`improvements.md`** — the full implementation guide. Every rewrite, every structural fix,
  every schema hint, in priority order. Used after the client is won.

---

## Phase 0 — Environment Setup

Install dependencies before any crawl begins. Always install silently and confirm before
proceeding.

```bash
pip install playwright requests beautifulsoup4 lxml reportlab --break-system-packages
python -m playwright install chromium
```

Verify Playwright is available:

```python
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    browser = p.chromium.launch()
    browser.close()
print("Playwright OK")
```

If the install fails, report the error and stop. Do not attempt to crawl without a working
headless browser.

---

## Phase 1 — Seed and Crawl

### 1.1 Accept the URL

The user provides a starting URL. Normalise it:

```python
from urllib.parse import urlparse, urljoin, urldefrag

def normalise(url, base):
    url, _ = urldefrag(url)          # strip fragments
    url = urljoin(base, url)         # resolve relative paths
    parsed = urlparse(url)
    return parsed._replace(query="").geturl()  # strip query strings for dedup
```

Extract the base domain (e.g. `example.com`) from the seed URL. Only follow links whose
`netloc` matches — no external crawling.

### 1.2 Crawl with Playwright

Use a breadth-first queue. Cap at 200 pages. Skip URLs matching any of:

- file extensions: `.pdf`, `.jpg`, `.jpeg`, `.png`, `.gif`, `.svg`, `.webp`, `.zip`,
  `.mp4`, `.mp3`, `.woff`, `.woff2`, `.css`, `.js`
- paths containing: `/wp-admin`, `/login`, `/logout`, `/cart`, `/checkout`, `#`
- URLs already visited

```python
from playwright.sync_api import sync_playwright
from collections import deque
from urllib.parse import urlparse

visited = set()
queue = deque([seed_url])
pages_data = []   # list of dicts, one per page

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    context = browser.new_context(
        user_agent="Mozilla/5.0 (compatible; SEOAuditBot/1.0)"
    )
    page = context.new_page()

    while queue and len(visited) < 200:
        url = queue.popleft()
        if url in visited:
            continue
        visited.add(url)

        try:
            response = page.goto(url, wait_until="domcontentloaded", timeout=20000)
            status = response.status if response else 0
            html = page.content()

            # Extract internal links
            links = page.eval_on_selector_all(
                "a[href]", "els => els.map(e => e.href)"
            )
            for link in links:
                norm = normalise(link, url)
                parsed = urlparse(norm)
                if parsed.netloc == base_domain and norm not in visited:
                    queue.append(norm)

            pages_data.append({
                "url": url,
                "status": status,
                "html": html,
            })

        except Exception as e:
            pages_data.append({
                "url": url,
                "status": 0,
                "error": str(e),
                "html": "",
            })

    browser.close()
```

### 1.3 Extract Copy and Metadata Per Page

For each page in `pages_data`, parse with BeautifulSoup:

```python
from bs4 import BeautifulSoup

def extract_page(data):
    soup = BeautifulSoup(data["html"], "lxml")

    # Remove nav, footer, script, style, cookie banners
    for tag in soup(["nav", "footer", "script", "style",
                     "noscript", "iframe", "svg"]):
        tag.decompose()

    title_tag = soup.find("title")
    meta_desc = soup.find("meta", attrs={"name": "description"})
    meta_robots = soup.find("meta", attrs={"name": "robots"})
    canonical = soup.find("link", attrs={"rel": "canonical"})
    og_title = soup.find("meta", attrs={"property": "og:title"})
    og_desc = soup.find("meta", attrs={"property": "og:description"})
    og_image = soup.find("meta", attrs={"property": "og:image"})

    h1s = [h.get_text(strip=True) for h in soup.find_all("h1")]
    h2s = [h.get_text(strip=True) for h in soup.find_all("h2")]
    h3s = [h.get_text(strip=True) for h in soup.find_all("h3")]

    # Images: src + alt
    images = [
        {"src": img.get("src", ""), "alt": img.get("alt", None)}
        for img in soup.find_all("img")
    ]

    # CTAs: buttons and links with action-style text
    ctas = []
    for el in soup.find_all(["button", "a"]):
        text = el.get_text(strip=True)
        if text and len(text.split()) <= 10:
            ctas.append(text)

    # Body word count (visible text)
    body_text = soup.get_text(separator=" ", strip=True)
    word_count = len(body_text.split())

    # Schema.org markup
    schema_tags = [
        s.string for s in soup.find_all("script", attrs={"type": "application/ld+json"})
        if s.string
    ]

    return {
        "url": data["url"],
        "status": data.get("status", 0),
        "title": title_tag.get_text(strip=True) if title_tag else "",
        "meta_description": meta_desc["content"] if meta_desc and meta_desc.get("content") else "",
        "robots": meta_robots["content"] if meta_robots and meta_robots.get("content") else "",
        "canonical": canonical["href"] if canonical and canonical.get("href") else "",
        "og_title": og_title["content"] if og_title and og_title.get("content") else "",
        "og_desc": og_desc["content"] if og_desc and og_desc.get("content") else "",
        "og_image": og_image["content"] if og_image and og_image.get("content") else "",
        "h1s": h1s,
        "h2s": h2s,
        "h3s": h3s,
        "images": images,
        "ctas": ctas,
        "word_count": word_count,
        "schema_json": schema_tags,
        "body_text_sample": body_text[:2000],
    }
```

Run `extract_page` on every item in `pages_data` and store results in `pages`.

---

## Phase 2 — Infer Site Context

Before scoring anything, infer product context from the homepage copy:

1. **Brand name** — from `<title>` or the first H1 on `/`
2. **Product category** — from H1, H2s, and meta description on `/`
3. **Primary differentiator** — any specific claims, numbers, or feature callouts
4. **Tone register** — classify as formal, casual, technical, or friendly based on body
   copy vocabulary and sentence length
5. **Product layer terms** — brand name, feature names, proprietary terminology found in
   copy
6. **Category layer terms** — market category phrases found in copy

Document these before scoring. Every score and every issue description depends on them.

If the homepage yields no usable context (blank page, JS-only with no SSR), note this and
flag the site as potentially unauditable without server-side rendering access.

---

## Phase 3 — Audit Every Page

For every page in `pages`, run all checks below. Produce one `page_audit` dict per page.

### 3.1 Title Tag

| Check | Pass condition |
|-------|---------------|
| Present | `title` not empty |
| Length | 50–60 characters |
| Primary keyword | product-layer or category-layer term in first 60 chars |
| Unique | no duplicate title across other pages |
| Branded | brand name appears (homepage or key pages) |

Score: 0–10. Deduct 2 per failed check.

### 3.2 Meta Description

| Check | Pass condition |
|-------|---------------|
| Present | `meta_description` not empty |
| Length | 120–158 characters |
| Keyword present | at least one keyword layer represented |
| CTA or benefit | contains an action phrase or value claim |
| Unique | not duplicated across pages |

Score: 0–10.

### 3.3 H1

| Check | Pass condition |
|-------|---------------|
| Exactly one H1 | `len(h1s) == 1` |
| Primary keyword early | keyword in first 60 chars |
| Not duplicate of title | H1 ≠ exact title string |
| Length | under 70 characters |

Score: 0–10.

### 3.4 Heading Hierarchy

Walk H1 → H2 → H3 in order. Flag:

- H3 appears before any H2 on the page
- Multiple H1s
- Zero H1
- H2s that skip directly from H1 (not an issue by itself, but note if H3s exist without
  parent H2s)

Issue severity: High if zero H1 or multiple H1s. Medium otherwise.

### 3.5 Images

For every image:

- `alt` attribute missing entirely → High priority
- `alt` is empty string (`""`) on a non-decorative image → Medium priority (decorative
  images with empty alt are correct — flag only where the image appears meaningful)
- `alt` text is generic: "image", "photo", "img", numbered filenames → Medium priority
- `alt` over 125 characters → Low priority

### 3.6 GEO Entity Clarity

Score 0–10 based on how well the page copy enables an LLM to understand and cite the page:

- 9–10: Brand name + category + use case + differentiator all appear
- 6–8: Two or three elements present
- 3–5: Generic copy; LLM cannot classify
- 0–2: No entity signal

Score from `body_text_sample`, H1, and meta description.

### 3.7 Keyword Coverage

- Does the page use at least one product-layer term? (Y/N)
- Does the page use at least one category-layer term? (Y/N)

A page with neither: critical gap. A page with only one layer: medium gap.

### 3.8 Content Depth

- Under 300 words → thin content, flag High
- 300–600 words → borderline, flag Medium
- Over 600 words → acceptable (no flag)

Use `word_count`.

### 3.9 CTA Quality

For each CTA string captured:

| Score | Criteria |
|-------|---------|
| 8–10 | Action verb + specific object + benefit |
| 5–7 | Clear action, no benefit |
| 2–4 | Vague ("click here", "learn more", "submit") |
| 0–1 | No action signal |

If no CTAs detected: flag the page as missing conversion elements.

### 3.10 Technical Flags

| Check | Flag if failing |
|-------|----------------|
| HTTP status not 200 | 4xx or 5xx pages |
| No canonical tag | Flag Medium |
| Canonical mismatch | Canonical points to a different URL than the page URL |
| Robots noindex | Flag — page excluded from search |
| No OG title or OG description | Flag Low (social sharing quality) |
| No OG image | Flag Low |
| No structured data (schema) | Flag Medium — GEO signal missing |
| Duplicate title | Flag High |
| Duplicate meta description | Flag High |

### 3.11 Page-Level Score

Composite: average of Title (25%), Meta Description (15%), H1 (20%), GEO Clarity (20%),
Keyword Coverage (10%), CTA Quality (10%).

Overall site score: weighted average across all pages, weighting homepage at 2×.

---

## Phase 4 — Aggregate Site-Level Issues

After all pages are scored, identify cross-site patterns:

1. **Duplicate titles** — list all pairs
2. **Duplicate meta descriptions** — list all pairs
3. **Orphan pages** — pages with no internal links pointing to them (from the crawl graph)
4. **Redirect chains** — if any URLs in the crawl returned 301/302, note the chain
5. **Broken internal links** — any internal URL returning 4xx/5xx
6. **Pages with no outbound internal links** — dead ends
7. **Coverage gaps** — product-layer terms absent from more than 50% of pages
8. **Schema gaps** — no `SoftwareApplication`, `Organization`, or `Product` schema found
   anywhere on the site
9. **Mobile/viewport** — check `<meta name="viewport">` present on all pages (flag if
   missing)

---

## Phase 5 — Write `improvements.md`

This file is for internal use after the client is won. It is the agent's implementation
guide. Structure it so an AI coding agent can apply every fix directly.

```markdown
# Improvements — [Site Name]
**URL:** [seed URL]
**Audit date:** [date]
**Total pages crawled:** [n]
**Site score:** [X/10]

---

## How to Use This File

Work through sections in order. Each fix includes the current value, the problem, and the
exact replacement. Apply fixes to the live CMS, codebase, or via the platform's SEO
settings panel.

---

## Priority 1 — Critical (Fix First)

### [Page URL or Site-Wide]

**Issue:** [specific problem]
**Location:** [title tag / H1 / meta description / image alt / etc.]

**Current:**
> [exact current text]

**Replace with:**
> [exact rewrite]

**Why:** [1–2 sentences: keyword added, layer served, constraint respected]

---

[Repeat for each Critical issue]

---

## Priority 2 — High Impact

[Same format]

---

## Priority 3 — Medium

[Same format]

---

## Priority 4 — Low / Optional

[Same format]

---

## Structural Fixes

### Heading Hierarchy Corrections
[List each page with broken hierarchy, current structure, corrected structure]

### Schema Markup to Add
[For each page: which schema type, which fields to populate, example JSON-LD block]

### Technical Fixes
[Canonical tags to add, OG tags to add, viewport tags, redirect chains to resolve]

---

## Rewritten Copy Blocks

[For every page, in priority order: original copy → rewritten copy with rationale.
Follow the same CTA formula and dual-layer keyword rules as the SEO + GEO skill.]

### CTA Formula
Every CTA rewrite: action verb + specific object + implicit or explicit benefit.
- "Learn more" → "See how [category] works for [user type]"
- "Get started" → "Start your free trial — no credit card required"
- "Contact us" → "Talk to the team — reply within one business day"

### Rewrite Constraints
1. No factual changes.
2. No invented data or statistics.
3. Tone register preserved: [detected tone].
4. Legal and disclaimer language untouched.

---

## Score Summary

| Page | Title | Meta Desc | H1 | GEO | Keywords | CTA | Overall |
|------|-------|-----------|-----|-----|----------|-----|---------|
| / Homepage | X | X | X | X | X | X | X |
| /about | ... |
```

Write every fix explicitly. No vague "improve the H1" — always show the exact replacement.

---

## Phase 6 — Generate `audit_report.pdf`

This is the client pitch. Professional, visual, persuasive. Findings and issues only. No
rewrites. No fix instructions. The goal: make the client feel the gap between where they
are and where they could be, and position you as the person who can close it.

### 6.1 PDF Generation

Use `reportlab` with `SimpleDocTemplate` and `Platypus`. Never use `canvas` directly for
multi-page documents.

```python
from reportlab.lib.pagesizes import A4
from reportlab.platypus import (
    SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle,
    HRFlowable, PageBreak
)
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib import colors
from reportlab.lib.units import mm
from reportlab.lib.enums import TA_CENTER, TA_LEFT

doc = SimpleDocTemplate(
    "/mnt/user-data/outputs/audit_report.pdf",
    pagesize=A4,
    leftMargin=20*mm,
    rightMargin=20*mm,
    topMargin=20*mm,
    bottomMargin=20*mm,
)
```

### 6.2 Colour Palette

Define once and reuse:

```python
BRAND_DARK    = colors.HexColor("#1a1a2e")
BRAND_MID     = colors.HexColor("#16213e")
ACCENT        = colors.HexColor("#e94560")
ACCENT_LIGHT  = colors.HexColor("#f5a623")
NEUTRAL_LIGHT = colors.HexColor("#f4f4f8")
TEXT_BODY     = colors.HexColor("#2c2c3e")
SCORE_RED     = colors.HexColor("#e94560")
SCORE_AMBER   = colors.HexColor("#f5a623")
SCORE_GREEN   = colors.HexColor("#27ae60")

def score_colour(score):
    if score < 5:
        return SCORE_RED
    if score < 7.5:
        return SCORE_AMBER
    return SCORE_GREEN
```

### 6.3 Custom Styles

```python
styles = getSampleStyleSheet()

styles.add(ParagraphStyle(
    "Cover_Title",
    fontName="Helvetica-Bold",
    fontSize=36,
    leading=42,
    textColor=colors.white,
    alignment=TA_CENTER,
))
styles.add(ParagraphStyle(
    "Cover_Sub",
    fontName="Helvetica",
    fontSize=14,
    leading=20,
    textColor=colors.HexColor("#ccccdd"),
    alignment=TA_CENTER,
))
styles.add(ParagraphStyle(
    "Section_Heading",
    fontName="Helvetica-Bold",
    fontSize=18,
    leading=24,
    textColor=BRAND_DARK,
    spaceBefore=12,
    spaceAfter=6,
))
styles.add(ParagraphStyle(
    "Body",
    fontName="Helvetica",
    fontSize=10,
    leading=15,
    textColor=TEXT_BODY,
    spaceAfter=6,
))
styles.add(ParagraphStyle(
    "Caption",
    fontName="Helvetica-Oblique",
    fontSize=9,
    leading=12,
    textColor=colors.HexColor("#888899"),
))
styles.add(ParagraphStyle(
    "Issue_High",
    fontName="Helvetica-Bold",
    fontSize=10,
    leading=14,
    textColor=SCORE_RED,
))
styles.add(ParagraphStyle(
    "Issue_Medium",
    fontName="Helvetica-Bold",
    fontSize=10,
    leading=14,
    textColor=SCORE_AMBER,
))
```

### 6.4 PDF Structure

Build the `story` list in this order:

**Page 1 — Cover**

Dark background block (simulate with a full-width coloured Table). Include:

- Audit title: "Website SEO + GEO Audit"
- Client website domain
- Prepared by: [your name if known, else "SEO Audit Report"]
- Date

To create a coloured cover block, use a single-cell Table with background colour:

```python
cover_table = Table(
    [[Paragraph("Website SEO + GEO Audit", styles["Cover_Title"])]],
    colWidths=[170*mm],
    rowHeights=[60*mm],
)
cover_table.setStyle(TableStyle([
    ("BACKGROUND", (0, 0), (-1, -1), BRAND_DARK),
    ("VALIGN", (0, 0), (-1, -1), "MIDDLE"),
    ("ALIGN", (0, 0), (-1, -1), "CENTER"),
    ("LEFTPADDING", (0, 0), (-1, -1), 10),
    ("RIGHTPADDING", (0, 0), (-1, -1), 10),
]))
story.append(cover_table)
```

**Page 2 — Executive Summary**

3–5 sentences. Cover:
- Total pages audited
- Overall site score (X/10) with a one-line verdict
- Top 3 critical findings (named, not generic)
- One sentence on the opportunity ("Sites that fix these issues typically see…")

Do not list solutions here. Frame everything as findings.

**Page 3 — Scoring Overview**

A summary table: one row per page, columns for each dimension score and overall. Use
`score_colour` to colour the Overall cell.

```python
score_table_data = [["Page", "Title", "Meta", "H1", "GEO", "Keywords", "CTA", "Overall"]]
for p in sorted_pages:
    row = [
        p["url_short"],
        str(p["title_score"]),
        str(p["meta_score"]),
        str(p["h1_score"]),
        str(p["geo_score"]),
        str(p["kw_score"]),
        str(p["cta_score"]),
        str(p["overall"]),
    ]
    score_table_data.append(row)

score_table = Table(score_table_data, repeatRows=1)
score_table.setStyle(TableStyle([
    ("BACKGROUND", (0, 0), (-1, 0), BRAND_DARK),
    ("TEXTCOLOR", (0, 0), (-1, 0), colors.white),
    ("FONTNAME", (0, 0), (-1, 0), "Helvetica-Bold"),
    ("FONTSIZE", (0, 0), (-1, -1), 8),
    ("ROWBACKGROUNDS", (0, 1), (-1, -1), [NEUTRAL_LIGHT, colors.white]),
    ("GRID", (0, 0), (-1, -1), 0.5, colors.HexColor("#ccccdd")),
    ("ALIGN", (1, 0), (-1, -1), "CENTER"),
    ("VALIGN", (0, 0), (-1, -1), "MIDDLE"),
    ("TOPPADDING", (0, 0), (-1, -1), 4),
    ("BOTTOMPADDING", (0, 0), (-1, -1), 4),
]))
# Colour the Overall column by score
for i, p in enumerate(sorted_pages, start=1):
    score_table.setStyle(TableStyle([
        ("BACKGROUND", (-1, i), (-1, i), score_colour(p["overall"])),
        ("TEXTCOLOR", (-1, i), (-1, i), colors.white),
    ]))
story.append(score_table)
```

**Page 4+ — Issues by Category**

Group issues into sections. For each section, use a heading and a list of findings. Each
finding states: page URL, what was found, why it matters. Never state the fix.

Sections (use only sections where issues were found):

1. Title Tag Issues
2. Meta Description Issues
3. Heading Structure Issues
4. Image Accessibility Issues
5. Content Depth Issues
6. GEO Visibility Gaps
7. CTA Weakness
8. Technical Issues (broken links, missing canonicals, noindex, etc.)
9. Cross-Site Patterns (duplicates, orphan pages, schema gaps)

For High priority issues, use red accent. Medium use amber. Low use grey.

```python
issue_style = styles["Issue_High"] if issue["priority"] == "High" else styles["Issue_Medium"]
story.append(Paragraph(f"⚠ {issue['page']} — {issue['description']}", issue_style))
story.append(Paragraph(issue["why_it_matters"], styles["Body"]))
story.append(Spacer(1, 4))
```

**Last Page — Next Steps**

Frame as a conversation opener, not a price list:

```
What Happens Next

This audit identifies [N] issues across [N] pages. Addressing them will improve your
site's visibility in both traditional search and AI-generated answers.

The next step is a 30-minute call to walk through the findings, answer your questions,
and agree on priorities. No commitment required.

[Contact name / email / calendar link if provided by user — otherwise leave a
placeholder: "→ [Your contact details]"]
```

### 6.5 Build and Save

```python
doc.build(story)
print("PDF written to /mnt/user-data/outputs/audit_report.pdf")
```

### 6.6 PDF Quality Checks

Before presenting the file, verify:

- [ ] File size > 0 bytes
- [ ] No Python exceptions during `doc.build()`
- [ ] Cover page has site domain visible
- [ ] Score table has at least the homepage row
- [ ] "Next Steps" page is the final page

If any check fails, print the error and re-run `doc.build()` with simplified content
(drop tables if table rendering failed, fall back to paragraph lists).

---

## Phase 7 — Output

Present both files using `present_files`:

```
audit_report.pdf — client-ready pitch, [N] pages audited, overall score [X/10]
improvements.md  — full fix list, [N] critical issues, [N] high priority
```

After presenting files, give the user a 3-sentence summary:
- What the site's biggest weakness is
- What the overall score was
- What to do next (send the PDF, run the fixes after the call)

---

## Phase 8 — Edge Cases

**JavaScript-only pages (no SSR)**: Playwright executes JS, so most SPAs will render. If
`body_text_sample` is empty or under 50 words after JS execution, add `page.wait_for_load_state("networkidle")` and retry once. If still empty, flag the page as "JS-rendered — copy not extracted" in both output files.

**Password-protected pages**: Skip and note in both files that those pages were excluded.

**Very large sites (200+ pages)**: Crawl cap is 200. Note in the report that a full audit
covered the first 200 pages discovered and that additional pages may exist.

**Robots.txt blocking**: Check `robots.txt` before crawling. If the audit bot is
disallowed, note this and proceed anyway (the audit is for the owner) — but flag it in
the report as a sign the site's crawl directives should be reviewed.

**HTTP errors on seed URL**: If the seed URL returns non-200, stop immediately and tell
the user the site appears to be down or the URL is incorrect.

**Redirects**: Follow up to 5 redirects. Log the chain. Flag redirect chains longer than
2 hops as a technical issue.

**Timeout**: If more than 30% of pages time out, note this in the report under Technical
Issues. Do not block report generation.

---

## Write-Like-a-Human Rules for All Written Output

Apply to every sentence in `improvements.md`, the PDF, and any agent-facing output.

**Banned words:** delve into, underscore, pivotal, realm, harness, illuminate, bolster,
streamline, facilitate, foster, leverage, cutting-edge, innovative, revolutionary,
game-changing, seamless, robust, scalable, holistic, synergy, paradigm shift, nuanced,
comprehensive, best practices, Certainly, Absolutely, Of course, Great question,
I'd be happy to, That being said, At its core, To put it simply, It's worth noting,
In today's world, generally speaking, various, a number of, truly, literally, essentially,
ultimately, clearly, obviously, fascinating, incredible, amazing, utilize, commence.

**Structural rules:**
- Audit notes and issue descriptions in prose, not bullet fragments.
- Never open with "It is important to note."
- Name the specific problem: "The H1 on /pricing contains no category keyword" beats
  "the heading could be improved."

**AI sentence patterns to avoid:**
- "Not a weakness, but an opportunity"
- "Whether you are a beginner or expert…"
- "In an era where users expect…"
- "[Site] allows you to unlock…"

**Self-check:** If a sentence could appear word-for-word in a report about a different
website, rewrite it until it could not.
