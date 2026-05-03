# Schema Check Rules

The 12 check rule sets plus the meta-check for malformed JSON-LD. Each rule defines: when to fire, severity, finding title, detail, recommendation, and lever.

**Read this file before running the audit.** Do not infer rules — apply them as written. Severity and lever assignments are deliberate and consistent with the Rush Media free Schema Detector tool.

---

## Table of contents

0. [Meta — Invalid JSON-LD](#0-meta--invalid-json-ld)
1. [Organization](#1-organization)
2. [WebSite](#2-website)
3. [BreadcrumbList](#3-breadcrumblist)
4. [Article](#4-article)
5. [Product](#5-product)
6. [LocalBusiness](#6-localbusiness)
7. [SoftwareApplication](#7-softwareapplication)
8. [Person](#8-person)
9. [Review / AggregateRating](#9-review--aggregaterating)
10. [Event](#10-event)
11. [VideoObject](#11-videoobject)
12. [FAQPage](#12-faqpage)
13. [HowTo](#13-howto)

---

## 0. Meta — Invalid JSON-LD

**Fires when:** one or more JSON-LD blocks failed to parse.

**Severity:** P0
**Lever:** Lever 1 — Crawl & Indexation
**schemaType in finding:** `Organization` (used as a generic bucket so this finding sorts to the top)

**Title:** `N JSON-LD blocks failed to parse` (or `1 JSON-LD block failed to parse`)

**Detail:** One or more `<script type="application/ld+json">` blocks contain malformed JSON. Google Search Console will report these as errors and the markup contributes nothing until fixed.

**Recommendation:** Validate every JSON-LD block with Google's Rich Results Test (search.google.com/test/rich-results) and fix syntax errors. Common causes: trailing commas, unquoted property names, smart quotes injected by a CMS, unescaped quotes in string values.

---

## 1. Organization

**Lever:** Lever 6 — Authority & GEO Visibility

### 1.1 Missing entirely

| Page type | Severity |
|-----------|----------|
| homepage | P0 |
| anything else | P1 |

**Title (homepage):** `Organization schema missing on homepage`
**Title (other):** `Organization schema not detected site-wide`

**Detail (homepage):** Your homepage has no Organization markup. Search engines and LLMs use this to anchor your brand entity in their knowledge graph — without it, your brand has weaker entity signals for AI search and rich results.

**Detail (other):** No Organization markup found on this page. While not strictly required on every page, Organization schema in the global header improves entity consistency across the site.

**Recommendation:** Add Organization JSON-LD with `name`, `url`, `logo`, and `sameAs` links to verified social and Wikipedia profiles.

### 1.2 Missing required properties

Required: `name`, `url`.

**Severity:** P0
**Title:** `Organization missing required: {missing properties joined by ", "}`
**Detail:** Google requires name and url for Organization schema to be eligible for rich results and knowledge panel features.
**Recommendation:** Add the missing properties.

### 1.3 Missing logo

**Severity:** P2
**Title:** `Organization schema missing logo`
**Detail:** Logo is a Google-recommended property. Without it, your brand's visual identity won't appear in knowledge panels or rich results.
**Recommendation:** Add `logo` as an ImageObject with a square or near-square image at minimum 112x112 pixels.

### 1.4 Missing sameAs

**Severity:** P2
**Title:** `Organization missing sameAs links`
**Detail:** sameAs links to verified social profiles, Wikipedia, and Wikidata are primary entity-disambiguation signals. LLMs cross-reference these when deciding which entity to cite.
**Recommendation:** Add a `sameAs` array including LinkedIn, X, Wikipedia, Wikidata, and other authoritative profiles for the entity.

---

## 2. WebSite

**Lever:** Lever 6 — Authority & GEO Visibility

### 2.1 Missing entirely

| Page type | Severity |
|-----------|----------|
| homepage | P1 |
| anything else | P2 |

**Title:** `WebSite schema not detected`
**Detail:** WebSite schema declares the canonical site identity and (with `potentialAction`) enables the sitelinks search box for branded searches.
**Recommendation:** Add WebSite JSON-LD with `@id`, `url`, `name`, and a `potentialAction` of type `SearchAction` with a target template like `https://example.com/search?q={search_term_string}`.

### 2.2 Missing potentialAction (SearchAction)

**Severity:** P2
**Title:** `WebSite missing SearchAction`
**Detail:** Your WebSite schema doesn't declare a `potentialAction`. This is what tells Google your site has internal search and qualifies you for the sitelinks search box on branded SERPs.
**Recommendation:** Add `potentialAction` as a `SearchAction` with `target` as an `EntryPoint` object pointing to your search results URL with `{search_term_string}` interpolation.

---

## 3. BreadcrumbList

**Lever:** Lever 2 — Canonical Architecture

### 3.1 Missing entirely

Skip on homepage (breadcrumbs on the homepage itself are unusual).

| Page type | Severity |
|-----------|----------|
| homepage | _do not fire_ |
| pages with depth ≥1 (URL has any path segment) | P1 |
| other | P2 |

**Title:** `BreadcrumbList schema missing`
**Detail:** Breadcrumb structured data improves the URL display in SERPs (replacing the raw URL with a human-readable path) and helps LLMs understand site hierarchy when deciding what to cite.
**Recommendation:** Add BreadcrumbList JSON-LD with `itemListElement` entries for each level from home to the current page. Each entry needs `position`, `name`, and `item` (URL).

### 3.2 itemListElement empty or missing

**Severity:** P0
**Title:** `BreadcrumbList has no itemListElement entries`
**Detail:** BreadcrumbList schema is present but contains no breadcrumb items, which means it provides no value and may be flagged in Search Console.
**Recommendation:** Populate `itemListElement` with `ListItem` entries containing `position` (1-indexed), `name`, and `item` (URL).

---

## 4. Article

**Lever:** Lever 4 — Content Production & Optimization

Applies to canonical type `Article` (which absorbs `NewsArticle`, `BlogPosting`, `TechArticle`, etc. via the alias table).

### 4.1 Missing entirely on an article page

Only fires when `pageType === "article"`.

**Severity:** P0
**Title:** `Article schema missing on article page`
**Detail:** This appears to be an article or blog post but has no Article-family schema. This blocks eligibility for Top Stories, weakens AI Overview citation candidacy, and removes author entity signals — a critical gap for content-led SEO and GEO.
**Recommendation:** Add Article (or NewsArticle / BlogPosting) JSON-LD with `headline`, `datePublished`, `dateModified`, `author` (as Person with `@id` and `url`), `publisher` (Organization), and `image`.

### 4.2 Missing required properties

Required: `headline`, `datePublished`, `author`.

**Severity:** P0
**Title:** `Article missing required: {missing properties}`
**Detail:** Google requires headline, datePublished, and author for Article schema to be eligible for Top Stories and standard rich results.
**Recommendation:** Add the missing properties.

### 4.3 Author is a string instead of a Person entity

**Severity:** P1
**Title:** `Article author is a plain string, not a Person entity`
**Detail:** Author as a string provides no entity linking. For E-E-A-T signaling and AI citation, author should be a Person object with `@id`, `name`, and `url` pointing to an author bio page.
**Recommendation:** Convert `author` to a Person object: `{ "@type": "Person", "@id": "https://yoursite.com/authors/jane-doe#Person", "name": "Jane Doe", "url": "https://yoursite.com/authors/jane-doe" }`. Lever override: this finding maps to **Lever 6 — Authority & GEO Visibility** (entity linking, not content production).

### 4.4 Missing dateModified

**Severity:** P2
**Title:** `Article missing dateModified`
**Detail:** dateModified signals freshness — important for evergreen content where the published date is old but the content has been updated.
**Recommendation:** Add `dateModified` as an ISO 8601 timestamp matching the last meaningful content update.

### 4.5 Missing image

**Severity:** P2
**Title:** `Article missing image`
**Detail:** Image is recommended for Article schema and required for Top Stories carousel eligibility on mobile.
**Recommendation:** Add `image` as an array of `ImageObject` (or URL strings) with at least one 16:9, 4:3, and 1:1 variant at minimum 1200px width.

### 4.6 Missing publisher

**Severity:** P2
**Title:** `Article missing publisher`
**Detail:** Publisher links the article back to the brand entity. Without it, the article floats free of organizational identity for AI search.
**Recommendation:** Add `publisher` as an Organization object with `name` and `logo` (logo as ImageObject, minimum 112px height).

---

## 5. Product

**Lever:** Lever 4 — Content Production & Optimization

### 5.1 Missing entirely on a product page

Only fires when `pageType === "product"`.

**Severity:** P0
**Title:** `Product schema missing on product page`
**Detail:** This appears to be a product page with no Product schema. This blocks every form of rich product result — price snippets, ratings, availability badges, and the merchant listings feed.
**Recommendation:** Add Product JSON-LD with `name`, `image`, `description`, `offers` (Offer with `price` + `priceCurrency` + `availability`), and `brand`. Add `aggregateRating` if you have customer reviews.

### 5.2 Missing required properties

Required: `name`, `image`.

**Severity:** P0
**Title:** `Product missing required: {missing properties}`
**Detail:** Product schema requires name and image at minimum. Without them, no rich product result will render.
**Recommendation:** Add the missing properties.

### 5.3 Missing offers

**Severity:** P1
**Title:** `Product missing offers`
**Detail:** Without offers, you can't show price or availability in SERPs — the most commercially valuable rich result data.
**Recommendation:** Add `offers` as an `Offer` (or `AggregateOffer`) with `price`, `priceCurrency`, `availability` (e.g., `https://schema.org/InStock`), and `url`.

### 5.4 Offer missing price or priceCurrency

When offers is present, validate the inner Offer object (or first item if array). Required: `price`, `priceCurrency`.

**Severity:** P1
**Title:** `Product offer missing: {missing properties}`
**Detail:** Offers must include both price and priceCurrency to qualify for price-bearing rich results.
**Recommendation:** Add the missing properties to the offers object.

### 5.5 Missing aggregateRating and review

**Severity:** P2
**Title:** `Product missing aggregateRating and review`
**Detail:** If this product has any customer reviews, surfacing them as `aggregateRating` unlocks star ratings in SERPs — significant CTR lift on commercial intent queries.
**Recommendation:** Add `aggregateRating` with `ratingValue`, `reviewCount`, `bestRating`. Only do this if you have genuine reviews — fabricated ratings violate Google policy.

### 5.6 Missing brand

**Severity:** P3
**Title:** `Product missing brand`
**Detail:** Brand is recommended and helps Google connect product schema to your Organization entity.
**Recommendation:** Add `brand` as an Organization or Brand object with `name`.

---

## 6. LocalBusiness

**Lever:** Lever 6 — Authority & GEO Visibility

### 6.1 Missing entirely on a local-business page

Only fires when `pageType === "local-business"`.

**Severity:** P0
**Title:** `LocalBusiness schema missing`
**Detail:** This page reads as a local business page but has no LocalBusiness markup. This is the primary input for local pack and Maps eligibility — without it you're invisible to local entity queries.
**Recommendation:** Add LocalBusiness JSON-LD with `name`, `address` (PostalAddress), `telephone`, `openingHoursSpecification`, and `geo` (GeoCoordinates).

### 6.2 Missing required properties

Required: `name`, `address`.

**Severity:** P0
**Title:** `LocalBusiness missing required: {missing properties}`
**Detail:** Google requires name and address (as PostalAddress) for LocalBusiness eligibility.
**Recommendation:** Add the missing properties.

### 6.3 Missing telephone

**Severity:** P2
**Title:** `LocalBusiness missing telephone`
**Detail:** Telephone is recommended and powers click-to-call in mobile rich results.
**Recommendation:** Add `telephone` in international format (e.g., +1-514-555-0100).

### 6.4 Missing opening hours

Either `openingHoursSpecification` or `openingHours` must be present.

**Severity:** P2
**Title:** `LocalBusiness missing opening hours`
**Detail:** Opening hours qualify you for hours-of-operation in the knowledge panel and local pack.
**Recommendation:** Add `openingHoursSpecification` as an array of `OpeningHoursSpecification` objects with `dayOfWeek`, `opens`, and `closes`.

### 6.5 Missing geo

**Severity:** P3
**Title:** `LocalBusiness missing geo coordinates`
**Detail:** Explicit geo coordinates remove ambiguity for Google's local matching.
**Recommendation:** Add `geo` as a `GeoCoordinates` object with `latitude` and `longitude`.

---

## 7. SoftwareApplication

**Lever:** Lever 4 — Content Production & Optimization

Applies to canonical type `SoftwareApplication` (which absorbs `MobileApplication`, `WebApplication`, `GameApplication` via the alias table). Critical for SaaS, native mobile apps, and any web-based application — historically underchecked but increasingly important for AI search citation of app-related queries.

### 7.1 Missing entirely on an app/SaaS page

Only fires when the page is identified as an app/SaaS page per §2.3 (URL = homepage, content includes app-store / download / sign-up signals).

**Severity:** P0
**Title:** `SoftwareApplication schema missing on app/SaaS page`
**Detail:** This page presents a software application (app, SaaS, or web app) but has no SoftwareApplication schema. This blocks rich results in app-related searches, weakens entity recognition for the app itself (separate from the company brand), and removes a primary citation hook for AI search engines answering app-comparison and recommendation queries.
**Recommendation:** Add SoftwareApplication JSON-LD with `name`, `applicationCategory` (e.g., "HealthApplication", "LifestyleApplication"), `operatingSystem` (e.g., "iOS, Android, Web"), `offers` (with price or "Free"), and `aggregateRating` if app-store ratings are public. Add `downloadUrl` pointing to the app store listings.

### 7.2 Missing required properties

When SoftwareApplication is present, required: `name`, `applicationCategory`, `operatingSystem`.

**Severity:** P0
**Title:** `SoftwareApplication missing required: {missing properties}`
**Detail:** Google requires name, applicationCategory, and operatingSystem for SoftwareApplication schema to be eligible for rich results.
**Recommendation:** Add the missing properties.

### 7.3 Missing offers

**Severity:** P1
**Title:** `SoftwareApplication missing offers`
**Detail:** Without offers, the app's pricing model is invisible to search and AI engines. Even free apps should declare this explicitly.
**Recommendation:** Add `offers` as an Offer object with `price` (or "0" for free), `priceCurrency`, and ideally `category` (e.g., "Subscription", "Freemium").

### 7.4 Missing aggregateRating

**Severity:** P2
**Title:** `SoftwareApplication missing aggregateRating`
**Detail:** App store ratings are a powerful citation signal. If the app has public ratings on the App Store or Google Play, surfacing them in SoftwareApplication schema unlocks star ratings in SERPs and improves AI search citation candidacy for "best app for X" queries.
**Recommendation:** Add `aggregateRating` with `ratingValue`, `ratingCount`, `bestRating`. Pull from your highest-rated platform; only use real ratings.

### 7.5 Missing downloadUrl

**Severity:** P3
**Title:** `SoftwareApplication missing downloadUrl`
**Detail:** downloadUrl explicitly links the schema to the app store listing, helping search engines connect the marketing page to the installable app.
**Recommendation:** Add `downloadUrl` as an array of URLs pointing to the App Store, Google Play, or direct download endpoints.

---

## 8. Person

**Lever:** Lever 6 — Authority & GEO Visibility

### 8.1 Missing entirely

Only fires when `pageType === "person"` or `pageType === "article"`.

| Page type | Severity |
|-----------|----------|
| person | P0 |
| article | P1 |

**Title:** `Person schema missing`

**Detail (person):** This appears to be an author bio or person page with no Person schema. This is the entity anchor for E-E-A-T and AI citation — without it, the author has no machine-readable identity.

**Detail (article):** Article pages benefit from author Person schema (either inline as the article's author, or as a separate Person object linked by `@id`).

**Recommendation:** Add Person JSON-LD with `name`, `url`, `image`, `jobTitle`, `sameAs` (LinkedIn, Wikipedia, ORCID, X), and a unique `@id` (e.g., the bio page URL with `#Person` fragment).

### 8.2 Missing required name

**Severity:** P0
**Title:** `Person missing name`
**Detail:** Person schema requires name as the bare minimum identifier.
**Recommendation:** Add the `name` property.

### 8.3 Missing sameAs

**Severity:** P2
**Title:** `Person missing sameAs`
**Detail:** sameAs is the disambiguation lifeline for Person entities. LLMs cross-reference these links to confirm identity and decide whose expertise to cite.
**Recommendation:** Add `sameAs` as an array including LinkedIn, X, Wikipedia, ORCID, university page, or any verifiable external profile.

### 8.4 Missing jobTitle and worksFor

Both must be missing for this finding to fire.

**Severity:** P3
**Title:** `Person missing jobTitle and worksFor`
**Detail:** Either jobTitle or worksFor (Organization) gives the person professional context — useful for E-E-A-T.
**Recommendation:** Add `jobTitle` as a string, and ideally `worksFor` as an Organization.

---

## 9. Review / AggregateRating

**Lever:** Lever 4 — Content Production & Optimization

This check fires when standalone Review or AggregateRating schema is detected. Product-nested ratings are handled by check 5.5; this check focuses on validation when review schema is present.

**Do not fire** "missing review schema" findings — absence of review schema is not itself a problem unless reviews are clearly present in the page content (which the audit can't reliably detect).

### 9.1 Review missing required properties

Required: `reviewRating`, `author`.

**Severity:** P1
**Title:** `Review missing required: {missing properties}`
**Detail:** Review schema requires reviewRating and author. Without them the markup is invalid for rich results.
**Recommendation:** Add the missing properties.

### 9.2 AggregateRating missing required properties

Required: `ratingValue`, and one of (`reviewCount`, `ratingCount`).

**Severity:** P1
**Title:** `AggregateRating missing required: {missing properties}`
**Detail:** AggregateRating requires ratingValue and reviewCount (or ratingCount) to be eligible for star ratings.
**Recommendation:** Add the missing properties.

---

## 10. Event

**Lever:** Lever 4 — Content Production & Optimization

### 10.1 Missing entirely on an event page

Only fires when `pageType === "event"`.

**Severity:** P0
**Title:** `Event schema missing on event page`
**Detail:** This appears to be an event page with no Event schema. This blocks eligibility for Google Events Search and event rich results.
**Recommendation:** Add Event JSON-LD with `name`, `startDate` (ISO 8601), `location` (Place or VirtualLocation), and `offers` (if ticketed).

### 10.2 Missing required properties

Required: `name`, `startDate`, `location`.

**Severity:** P0
**Title:** `Event missing required: {missing properties}`
**Detail:** Event requires name, startDate, and location for rich result eligibility.
**Recommendation:** Add the missing properties.

### 10.3 Missing endDate

**Severity:** P3
**Title:** `Event missing endDate`
**Detail:** endDate is recommended and helps with calendar-style listings.
**Recommendation:** Add `endDate` as an ISO 8601 timestamp.

### 10.4 Missing eventStatus

**Severity:** P3
**Title:** `Event missing eventStatus`
**Detail:** eventStatus signals scheduling state (e.g., `https://schema.org/EventScheduled`) and is increasingly expected after the COVID-era schema updates.
**Recommendation:** Add `eventStatus`, defaulting to `EventScheduled` if no special state applies.

---

## 11. VideoObject

**Lever:** Lever 4 — Content Production & Optimization

### 11.1 Missing entirely on a video page

Only fires when `pageType === "video"`.

**Severity:** P0
**Title:** `VideoObject schema missing on video page`
**Detail:** This page appears to host a video but has no VideoObject schema, blocking eligibility for video rich results and AI video citations.
**Recommendation:** Add VideoObject JSON-LD with `name`, `description`, `thumbnailUrl`, `uploadDate`, `contentUrl` or `embedUrl`, and `duration` (ISO 8601).

### 11.2 Missing required properties

Required: `name`, `description`, `thumbnailUrl`, `uploadDate`.

**Severity:** P0
**Title:** `VideoObject missing required: {missing properties}`
**Detail:** VideoObject requires name, description, thumbnailUrl, and uploadDate for rich result eligibility.
**Recommendation:** Add the missing properties.

### 11.3 Missing contentUrl and embedUrl

Both must be missing for this finding to fire.

**Severity:** P1
**Title:** `VideoObject missing contentUrl and embedUrl`
**Detail:** Either contentUrl or embedUrl is required so Google can verify the video is playable.
**Recommendation:** Add `contentUrl` pointing to the raw video file, or `embedUrl` pointing to the embeddable player.

### 11.4 Missing duration

**Severity:** P2
**Title:** `VideoObject missing duration`
**Detail:** Duration helps with key moments and is recommended for all video content.
**Recommendation:** Add `duration` in ISO 8601 format (e.g., `PT4M30S` for 4 minutes 30 seconds).

---

## 12. FAQPage

**Lever:** Lever 4 — Content Production & Optimization

**Do not fire** "missing FAQPage" findings — most sites should not have FAQPage schema. Only fire findings when FAQPage **is present**.

### 12.1 FAQPage has no questions

Fires when FAQPage is present but `mainEntity` is missing or empty.

**Severity:** P1
**Title:** `FAQPage has no questions`
**Detail:** FAQPage schema is present but mainEntity contains no Question entries — this is invalid markup.
**Recommendation:** Either populate `mainEntity` with `Question` objects (each with `acceptedAnswer`) or remove the FAQPage schema entirely.

### 12.2 FAQPage detected — deprecation note

Always fires when FAQPage is present and 11.1 did not fire.

**Severity:** P3
**Title:** `FAQPage detected — note: deprecated for most sites`
**Detail:** Google deprecated FAQPage rich results in August 2023 for most sites, retaining them only for government and authoritative health domains. The schema is still parsed by AI search engines and can support citation, but does not drive traditional rich results.
**Recommendation:** If your site qualifies for the retained FAQ rich result, keep it. Otherwise the schema is harmless to keep but should not be a priority — focus on Article and Person schema for AI citation instead.

---

## 13. HowTo

**Lever:** Lever 4 — Content Production & Optimization

**Do not fire** "missing HowTo" findings. Only fire when HowTo is present.

### 13.1 HowTo detected — deprecation note

Always fires when HowTo is present.

**Severity:** P3
**Title:** `HowTo detected — note: deprecated for rich results`
**Detail:** Google deprecated HowTo rich results in September 2023. The schema is still valid and parseable by AI search engines, but no longer renders as a step-by-step rich result in SERPs.
**Recommendation:** Schema is harmless to keep. If you want AI search visibility for procedural content, the more impactful path is well-structured Article schema with clear h2/h3 step headings — LLMs cite the markup-light version effectively.

---

## Helper notes for applying these checks

**"Property is present" means:** the property exists, is not `null`, is not an empty string, and is not an empty array. Whitespace-only strings count as missing.

**"Required" vs "recommended":** required properties produce P0 or P1 findings (depending on whether the schema itself is missing or just incomplete). Recommended properties produce P2 or P3.

**Multiple instances of the same schema:** if a page declares two `Article` JSON-LD blocks, run the Article check on the first one only. Note any duplication as a separate informational note in the response prose (not as a finding).

**Order of finding emission within a check:** emit "schema entirely missing" first, then "required properties missing", then recommended-property findings in the order listed above.
