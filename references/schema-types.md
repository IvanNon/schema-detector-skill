# Schema Type Aliases

Map raw `@type` values from JSON-LD into canonical keys used by the 12 checks. When a page declares `BlogPosting`, treat it as `Article` for the audit.

## Canonical keys (12)

These are the only valid values for `schemaType` in findings and `detectedTypeKeys` in the output:

```
Organization
WebSite
BreadcrumbList
Article
Product
LocalBusiness
FAQPage
HowTo
Person
Review
Event
VideoObject
```

## Alias map (raw → canonical)

```
Organization        → Organization
Corporation         → Organization
EducationalOrganization → Organization
NGO                 → Organization
GovernmentOrganization → Organization
Airline             → Organization

LocalBusiness       → LocalBusiness
Restaurant          → LocalBusiness
Store               → LocalBusiness
ProfessionalService → LocalBusiness
MedicalBusiness     → LocalBusiness
HomeAndConstructionBusiness → LocalBusiness
LegalService        → LocalBusiness
FinancialService    → LocalBusiness
Hotel               → LocalBusiness
LodgingBusiness     → LocalBusiness
DryCleaningOrLaundry → LocalBusiness

WebSite             → WebSite

BreadcrumbList      → BreadcrumbList

Article             → Article
NewsArticle         → Article
BlogPosting         → Article
TechArticle         → Article
ScholarlyArticle    → Article
Report              → Article
SocialMediaPosting  → Article
LiveBlogPosting     → Article

Product             → Product
ProductGroup        → Product
ProductModel        → Product
SomeProducts        → Product
IndividualProduct   → Product

FAQPage             → FAQPage

HowTo               → HowTo

Person              → Person

Review              → Review
AggregateRating     → Review
Rating              → Review

Event               → Event
BusinessEvent       → Event
EducationEvent      → Event
ExhibitionEvent     → Event
Festival            → Event
SocialEvent         → Event
SportsEvent         → Event

VideoObject         → VideoObject
Movie               → VideoObject
TVEpisode           → VideoObject
TVSeries            → VideoObject
```

## Special cases

- **`@graph` arrays** — flatten before mapping. Each object inside `@graph` is checked independently against this table.
- **Multiple `@type` values** — JSON-LD allows `"@type": ["Article", "TechArticle"]`. Take the first matching alias.
- **Subtypes not in this table** — record under the closest parent. `MedicalWebPage` not listed → treat as no canonical match (it's a `WebPage`, which we don't audit). `MobileApplication` not listed → no canonical match.
- **Synthetic `__INVALID_JSON_LD__`** — if a JSON-LD block fails to parse, record it with this synthetic type. The invalid JSON-LD meta-check (first check) handles it; the rest of the alias table doesn't apply.
