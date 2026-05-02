---
name: schema-detector
description: "Use this skill whenever the user wants to audit a single URL for structured data (schema.org markup). Trigger on phrases like: run schema detector, schema audit, audit schema, check schema, check structured data, audit JSON-LD, schema check, structured data audit, find schema gaps, audit my schema, schema findings, what schema is on this page, does this page have schema, validate my JSON-LD, schema for AI search, schema for AIO, schema for GEO. Also trigger when a user pastes a URL and asks 'is the schema correct', 'what's missing from my schema', 'audit this for structured data', 'check this page's schema', or 'run a quick schema check'. Do NOT use for full technical SEO audits (use tech-seo-audit skill instead), for AIO block generation (use ygramul-aio-cascade), or for crawling sitemaps / multiple URLs at once — this skill is single-URL only by design. Produces a deterministic JSON report with detected schema types, severity-ranked findings (P0/P1/P2/P3), and 7-Lever framework references — chainable into auto-research, Codex, and content briefs."
---

# Schema Detector

A single-URL structured data audit. Fetches a page, extracts JSON-LD / Microdata / RDFa, runs 12 schema-type checks, and returns a JSON report with severity-ranked findings.

**Output:** a single JSON object with detected schemas, all findings sorted P0→P3, and 7-Lever framework attribution per finding.

**Method:** this is the prompt-driven Claude-skill version of the Rush Media free Schema Detector tool. It mirrors the same 12 checks, severity logic, and Ygramul framing — but runs inside any Claude conversation and outputs machine-readable JSON for chaining.

**Companion to:** `tech-seo-audit` (full audit), `ygramul-aio-cascade` (Lever 4 AIO blocks). This skill is the focused Lever 6 / Lever 4 schema-only check.

---

## 1. Required inputs

| Input | Required | Default if missing |
|-------|----------|---------------------|
| **Target URL** | Yes | Ask the user |
| **Page type hint** | Optional | Inferred from URL + content + existing schema |

**Minimum viable input:** a single URL. Everything else is inferred.

If the user provides a URL but the page type is genuinely ambiguous (e.g., a thin homepage that could be a person bio), ask once for clarification before running the audit. Otherwise proceed.

---

## 2. Audit pipeline

Execute these steps in order. Do not skip steps.

### 2.1 Fetch the page

Use `web_fetch` on the target URL. If `web_fetch` returns empty or CDN-blocked content, fall back to `web_search` on the URL — search snippets and the result page often expose enough HTML for partial schema extraction. Note the fallback in the output.

### 2.2 Extract structured data

Scan the fetched HTML for three formats:

- **JSON-LD** — `<script type="application/ld+json">` blocks. This is the dominant format in 2025+; treat it as primary.
- **Microdata** — `[itemscope][itemtype]` attributes. Legacy but still present on older sites.
- **RDFa** — `[typeof]` attributes. Rare; check for completeness.

For JSON-LD, parse each block. Handle `@graph` arrays by flattening — every object inside `@graph` is treated as a top-level schema. If a block fails to parse (malformed JSON, trailing commas, smart quotes from a CMS), record it as a synthetic finding under `Organization` schema with severity `P0` titled "JSON-LD block failed to parse".

Map raw `@type` values to canonical keys using the alias table in `references/schema-types.md`. Multiple `@type` aliases collapse to one canonical key (e.g., `NewsArticle` and `BlogPosting` both → `Article`).

### 2.3 Infer page type

Decide what kind of page this is, in this priority order:

1. **Strongest signal — existing schema:** if `Product` is present → product page; if `Article` is present → article page; if `LocalBusiness` is present → local-business page; etc.
2. **URL path heuristics:** `/product/`, `/shop/`, `/p/` → product. `/blog/`, `/article/`, `/news/`, `/post/`, `/insights/` → article. `/event/` → event. `/video/` → video. `/author/`, `/team/` → person. Path is `/` or empty → homepage.
3. **OpenGraph signals:** `<meta property="og:type" content="article">` → article. `og:type` of `product`, `video`, `profile` → matching types.
4. **DOM signals:** large `<article>` element with multiple paragraphs → article. `[itemtype*="Product"]` → product.
5. **Default:** `generic`.

Page type **influences severity** for missing-schema findings: missing `Product` schema is `P0` on a product page but `P2` on a homepage. The check rules in §3 spell this out.

### 2.4 Run all 12 checks

For each canonical schema type below, run the corresponding check from `references/schema-checks.md`. Each check returns 0 or more findings.

The 12 checks plus 1 meta-check:

1. **Invalid JSON-LD meta-check** — runs first; surfaces malformed blocks
2. Organization
3. WebSite (with SearchAction)
4. BreadcrumbList
5. Article (incl. NewsArticle, BlogPosting)
6. Product
7. LocalBusiness
8. Person
9. Review / AggregateRating
10. Event
11. VideoObject
12. FAQPage (note: deprecated for most sites — flag accordingly)
13. HowTo (note: deprecated — flag accordingly)

Each finding must include:
- `schemaType` — one of the canonical keys
- `severity` — `P0` | `P1` | `P2` | `P3`
- `title` — short, ≤80 chars, action-oriented
- `detail` — 1–3 sentences explaining the impact
- `recommendation` — concrete fix, including specific properties to add
- `lever` — one of the 7 Levers (see §4)

### 2.5 Sort + summarize

Sort all findings by severity: `P0` first, then `P1`, `P2`, `P3`. Within the same severity, preserve check order from §2.4 (so `Invalid JSON-LD` always tops the list when present).

Compute the summary block:
- `totalDetectedTypes` — count of distinct canonical schema keys found
- `totalChecked` — always 12
- `counts` — `{P0, P1, P2, P3}` integer counts

### 2.6 Emit JSON

Output a single JSON object matching the schema in §5. Do not include any markdown commentary outside the JSON block. The user is chaining this into other tools.

---

## 3. Severity rubric

Severity is decided by **what the finding blocks**, not by how "big" the schema fix looks.

| Severity | Meaning | Example |
|----------|---------|---------|
| **P0 · Critical** | Breaks rich result eligibility, breaks parsing, or critical schema absent for this page type | Malformed JSON-LD; Product schema missing on a product page; Article missing required `headline` + `datePublished` + `author` |
| **P1 · High** | Required schema present but missing required properties; or important schema absent | Article author is a string instead of a Person entity; Product `offers` missing `price` |
| **P2 · Medium** | Recommended properties missing; optional schema gaps | Organization missing `logo`; WebSite missing `potentialAction` |
| **P3 · Low** | Best-practice additions; deprecated rich results that still parse | LocalBusiness missing `geo`; FAQPage detected (deprecated for most sites) |

Page-type modifier: missing schema on the **wrong** page is downgraded one or two severity levels. Missing schema on the **right** page is the canonical severity from `references/schema-checks.md`.

---

## 4. The 7-Lever framework

Every finding maps to one of these levers from the Ygramul methodology:

| Lever | When a finding maps here |
|-------|---------------------------|
| **Lever 1 — Crawl & Indexation** | Malformed JSON-LD that breaks parsing |
| **Lever 2 — Canonical Architecture** | BreadcrumbList issues — the schema closest to URL hierarchy |
| **Lever 4 — Content Production & Optimization** | Article, Product, Review, Event, VideoObject, FAQPage, HowTo — the page-content schemas |
| **Lever 6 — Authority & GEO Visibility** | Organization, WebSite, Person, LocalBusiness — the entity-anchor schemas that drive AI citation behavior |

Levers 3, 5, 7 are not used by this skill — they're keyword strategy, programmatic SEO, and analytics, all out of scope for a schema audit.

---

## 5. Output JSON shape

Emit exactly this shape. No additional fields, no missing fields. If a section has no entries, use an empty array `[]`.

```json
{
  "url": "https://example.com/page",
  "fetchedAt": "2026-05-02T14:30:00.000Z",
  "pageTitle": "Page title from <title> tag, or null if missing",
  "pageType": "homepage | article | product | local-business | event | video | person | generic",
  "fetchMethod": "web_fetch | web_search_fallback",
  "detectedSchemas": [
    {
      "type": "Article",
      "format": "json-ld",
      "rawTypeValue": "BlogPosting"
    }
  ],
  "detectedTypeKeys": ["Organization", "WebSite", "Article"],
  "findings": [
    {
      "schemaType": "Article",
      "severity": "P0",
      "title": "Article missing required: datePublished, author",
      "detail": "Google requires headline, datePublished, and author for Article schema to be eligible for Top Stories and standard rich results.",
      "recommendation": "Add the missing properties: datePublished as ISO 8601, author as a Person object with @id and url.",
      "lever": "Lever 4 — Content Production & Optimization"
    }
  ],
  "summary": {
    "totalDetectedTypes": 3,
    "totalChecked": 12,
    "counts": { "P0": 1, "P1": 0, "P2": 2, "P3": 0 }
  }
}
```

**Critical rules for the JSON output:**
- The JSON must be valid and parseable — no trailing commas, no comments, no non-standard syntax.
- Wrap the JSON in a single fenced ` ```json ` code block. Nothing before or after the block.
- If you need to explain something to the user, do it **before** the code block, not inside or after. Most users will paste this directly into another tool.
- `fetchedAt` is ISO 8601 with millisecond precision and `Z` timezone.
- `detectedSchemas[].rawTypeValue` is the original `@type` from the page (e.g., `NewsArticle`); `detectedSchemas[].type` is the canonical key (e.g., `Article`).

---

## 6. When to chain into other skills

After emitting the JSON, suggest follow-ups based on what was found:

| Pattern | Suggested next skill |
|---------|----------------------|
| ≥3 P0 findings | `tech-seo-audit` — bigger problems are likely |
| Article schema present but no AIO opener evident | `ygramul-aio-cascade` — generate one |
| Multiple sites to check, or sitewide schema policy needed | Recommend `tech-seo-audit` instead — this skill is single-URL only |

Phrase suggestions naturally: "Want me to run the full tech audit since I found 4 P0 issues?" not "INVOKE_SKILL: tech-seo-audit".

---

## 7. Failure modes and how to handle them

| Situation | Handling |
|-----------|----------|
| `web_fetch` returns 404 / 5xx | Tell the user the URL is unreachable, do not emit JSON, ask if they meant a different URL |
| `web_fetch` returns empty body but URL is valid | Try `web_search` on the URL as fallback. If that also fails, tell the user the page may be JS-rendered or bot-blocked and recommend they paste the rendered HTML directly |
| User pastes raw HTML instead of a URL | Skip step 2.1, treat the paste as the page content, set `url` to `null` and `fetchMethod` to `"user_paste"` |
| Page has zero schema | Still emit the full JSON. `detectedSchemas` is `[]`, `detectedTypeKeys` is `[]`, and the findings will be the page-type-appropriate "missing schema" findings (typically several P1s and P2s, plus P0s if it's a clearly-typed page like product or article) |
| Page has only valid schema with no findings | Still emit the full JSON. `findings` is `[]`, `counts` is all zeros. This is a legitimate clean result — say so plainly: "No findings — schema implementation is clean." |
| User asks to audit multiple URLs | This skill is single-URL only by design. Tell the user, recommend `tech-seo-audit` for sitewide work, and offer to run this skill once on the most important URL |

---

## 8. Reference files

The check rules and alias tables live in `references/`. Read them before running the audit if you don't already have them in working memory.

- `references/schema-types.md` — Alias table mapping raw `@type` values to canonical keys
- `references/schema-checks.md` — The 12 check rule sets (what to look for, what severity, what to recommend) — the heart of the skill

These files are loaded on demand. They're long because the rules are detailed; don't try to memorize them, just reference them when running the audit.
