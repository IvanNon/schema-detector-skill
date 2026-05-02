# Schema Detector

A Claude Skill that audits any URL for structured data — extracts every JSON-LD, Microdata, and RDFa block on the page, runs 12 schema-type checks, and returns a deterministic JSON report with severity-ranked findings (P0–P3) and 7-Lever framework attribution.

---

## The name

Schema is what tells robots what your page is *about*. Most pages get it wrong, get it broken, or skip it entirely. This skill is the detector — point it at a URL, find out what's missing, get the fix.

Built by [Ivan Nonveiller](https://www.linkedin.com/in/ivannonveiller/) — enterprise SEO/GEO strategist.

---

## What it does

You give it a URL. It returns a structured audit of every schema block on the page, plus a list of what's missing or broken.

- Fetches the target page (with `web_search` fallback for bot-blocked sites)
- Extracts JSON-LD, Microdata, and RDFa structured data
- Infers the page type (homepage, article, product, local-business, event, video, person)
- Runs 12 schema-type checks plus a meta-check for malformed JSON-LD
- Sorts findings by severity (P0 critical → P3 low)
- Maps every finding to one of the 7 Levers from the Ygramul methodology
- Returns: machine-readable JSON, chainable into other skills and audit workflows

---

## Why this instead of Google's Rich Results Test

The Rich Results Test answers "is this valid for rich results today?" — useful, but limited. It tells you nothing about AI search citation, entity anchoring, or whether the schema you have matches the page type Google thinks you are.

This skill audits for **what AI search engines and crawlers actually use**, not just what currently triggers a SERP rich result. Findings include:

- Author as a string vs. linked Person entity (E-E-A-T signal that survived the 2023 deprecations)
- Missing `sameAs` arrays on Organization and Person schema (the disambiguation lifeline LLMs cross-reference)
- WebSite schema without SearchAction (still expected; sitelinks search box still matters)
- Page-type mismatches (Article schema on a product page is its own kind of broken)

It also flags FAQPage and HowTo schema as P3 with deprecation context — present in Rich Results Test as "valid" but no longer driving rich results for most sites since 2023.

---

## The 12 checks

**Lever 6 — Authority & GEO Visibility (entity anchors)**
- Organization
- WebSite
- Person
- LocalBusiness

**Lever 4 — Content Production & Optimization (page schemas)**
- Article (incl. NewsArticle, BlogPosting)
- Product
- Review / AggregateRating
- Event
- VideoObject
- FAQPage *(flagged P3 — deprecated for most sites)*
- HowTo *(flagged P3 — deprecated)*

**Lever 2 — Canonical Architecture**
- BreadcrumbList

**Lever 1 — Crawl & Indexation (meta-check)**
- Invalid JSON-LD detection — surfaces malformed blocks as P0 before any other check runs

Severity is page-type-aware. Missing Product schema is P0 on a product page but P2 on a homepage. The full rule set lives in [`references/schema-checks.md`](./references/schema-checks.md).

---

## Install

1. Download the contents of this repo (or clone it)
2. Open [claude.ai](https://claude.ai) → click your profile → **Settings** → **Capabilities** → **Skills**
3. Click **Upload skill** and select the `SKILL.md` file (the references load automatically)
4. Done — the skill is now available in your account

**Requires** Claude Pro, Team, or Enterprise. Free tier doesn't support custom skills.

---

## Use

Start a new chat and type:

> Audit the schema on https://yourwebsite.com/your-page

Or any natural variant — "what schema is on this page", "run schema detector on...", "check structured data for...".

Claude will:

1. Fetch the page
2. Extract all structured data
3. Run the 12 checks
4. Return JSON with detected schemas, findings sorted P0→P3, and a summary block

If you'd rather have a Markdown report than JSON, ask for it explicitly. The default is JSON because that's what chains into other tools.

---

## Output shape

```json
{
  "url": "https://example.com/blog/post",
  "fetchedAt": "2026-05-02T14:30:00.000Z",
  "pageTitle": "Why Sleep Matters",
  "pageType": "article",
  "fetchMethod": "web_fetch",
  "detectedSchemas": [
    { "type": "Article", "format": "json-ld", "rawTypeValue": "BlogPosting" },
    { "type": "Organization", "format": "json-ld", "rawTypeValue": "Organization" }
  ],
  "detectedTypeKeys": ["Organization", "Article"],
  "findings": [
    {
      "schemaType": "Article",
      "severity": "P0",
      "title": "Article missing required: datePublished, author",
      "detail": "Google requires headline, datePublished, and author for Article schema to be eligible for Top Stories and standard rich results.",
      "recommendation": "Add datePublished as ISO 8601, author as a Person object with @id and url.",
      "lever": "Lever 4 — Content Production & Optimization"
    }
  ],
  "summary": {
    "totalDetectedTypes": 2,
    "totalChecked": 12,
    "counts": { "P0": 1, "P1": 0, "P2": 2, "P3": 0 }
  }
}
```

---

## Expected results

- Per-audit cost: ~$0.05–0.20 in Claude usage (covered by your Pro/Team/Enterprise plan)
- Audit time: 15–45 seconds depending on page size
- Output is deterministic — same URL produces the same finding shape every run, suitable for diffing audits over time

For schemas you fix:
- 1–2 weeks after shipping: Google re-crawls and re-validates
- 2–6 weeks: rich results begin appearing for newly-eligible schema (assuming Google approves)
- AI search citation effects (Perplexity, ChatGPT, AI Overviews) compound over months as entity signals strengthen

---

## When not to use

- **Sitewide audits** — this skill is single-URL by design. For full-site work, use a dedicated technical SEO audit tool or a sitemap-aware audit skill.
- **JS-rendered SPAs that inject schema client-side** — `web_fetch` can't execute JavaScript. The skill will undercount and tag the audit with `fetchMethod: "web_search_fallback"` so you know.
- **Pages still in draft** — it needs the published version to read.
- **Schema implementation** — this skill audits and recommends. It doesn't write the JSON-LD for you. Pair with a separate content brief or implementation skill.

---

## Companion skills

This is the focused single-page check. Two adjacent skills cover different scopes:

- **[ygramul-aio-cascade](https://github.com/IvanNon/ygramul-aio-cascade)** — Lever 4 AIO opener block generation. Run this *after* fixing schema gaps to capture AI Overview citations.
- **tech-seo-audit** — full-site technical audit covering all 7 Levers. Use when Schema Detector findings suggest deeper problems (3+ P0s, sitewide gaps).

---

## Contributing

Found a check rule that needs tightening? Severity assignment that doesn't match real-world impact? PRs welcome. Open an issue first for anything bigger than a typo fix.

The check rules live in `references/schema-checks.md` — edit there. The skill is designed to be tuned without touching `SKILL.md` itself.

---

## License

MIT. Use it, modify it, ship it in your own tools.

---

## About the author

**Ivan Nonveiller** — enterprise technical SEO and GEO strategist. 10+ years optimizing search for brands including Walmart, AutoTRADER, Ferrari, Maserati, Ritz-Carlton, Sotheby's, and BetterSleep. Based in Montréal.

- LinkedIn: [ivannonveiller](https://www.linkedin.com/in/ivannonveiller/)
- Ygramul methodology: [rushmediaagency.com](https://rushmediaagency.com)

This skill is the open-source version of Ygramul's Lever 6 (Authority) and Lever 4 (Content) entity-schema check system.
