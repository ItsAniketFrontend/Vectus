# SEO

The SEO team must be able to do all on-page work in the admin panel without a developer. This file covers what they control, how dynamic schema works, and the migration checklist.

## 1. What the SEO team controls

### Per page, product, collection, post or job (SEO tab)

| Field | Behaviour |
|---|---|
| Meta title | Live character + pixel-width counter; SERP preview (desktop & mobile). Empty → falls back to the meta template for that type |
| Meta description | Counter + preview; template fallback |
| URL slug | Editable; a change auto-creates a 301 |
| H1 | Defaults to the name/title; can be overridden. Templates enforce one H1 per page |
| Canonical URL | Defaults to self; override for duplicates |
| Robots | index/noindex, follow/nofollow |
| Open Graph / X card | Title, description, image (falls back to meta + main image) |
| Focus + secondary keywords | Used by the on-page checker |
| Breadcrumb label | Short label for breadcrumbs |
| Schema | Auto / Auto + custom / Custom only (see §2) |
| Sitemap | Include/exclude, priority |
| Image alt text | Per image in the media library; required on new uploads |
| Content | Full block editor: headings, body, FAQs, internal links, CTAs |

### Site-wide (SEO menu)

| Screen | What it does |
|---|---|
| Bulk editor | Spreadsheet-style grid of every URL: title, description, H1, canonical, robots, length warnings. Inline editing + CSV export/import |
| Meta templates | Fallback patterns per type, e.g. `{{product.name}} – {{attr.layers}} Layer Water Tank – Vectus` |
| Schema templates | JSON-LD templates per entity type (§2) |
| Redirects | Add/edit 301, 302 and 410 rules (exact, prefix or regex), CSV import, hit counter, chain & loop detection |
| 404 log | 404s seen on the live site, sorted by hits, with one-click "create redirect" |
| Sitemap & robots | Toggle entity types, edit robots.txt (guarded so staging stays noindex) |
| Tracking scripts | GTM, GA4, Search Console verification, Meta Pixel and others — head/body placement and consent category |
| SEO audit | Missing/duplicate titles & descriptions, titles too long/short, missing alt text, missing H1, noindex pages in the sitemap, broken internal links, orphan pages, thin content, redirect chains |
| Global | Default OG image, organisation details for schema, social profiles (`sameAs`), title separator, brand suffix |

### Template tokens

Usable in meta templates and schema templates:

`{{site.name}}` `{{site.url}}` `{{page.title}}` `{{page.url}}` `{{product.name}}` `{{product.description_plain}}` `{{product.image}}` `{{product.images}}` `{{product.brand}}` `{{product.sku}}` `{{product.capacity_range}}` `{{product.price_min}}` `{{product.price_max}}` `{{product.availability}}` `{{collection.name}}` `{{post.title}}` `{{post.author}}` `{{post.published_at}}` `{{job.title}}` `{{job.valid_through}}` `{{attr.<code>}}` (any product attribute, e.g. `{{attr.layers}}`)

## 2. Dynamic schema (JSON-LD)

Three layers, resolved by the backend per URL and returned in `seo.json_ld`:

1. **Auto schema** — built from data, so it is always correct and in sync (e.g. Product from the product and its variants, BreadcrumbList from the category path, FAQPage from FAQ blocks).
2. **Schema templates** — admin-editable JSON-LD with tokens, per entity type. Lets the SEO team add properties (e.g. `material`, `additionalProperty`) to every product at once.
3. **Custom per-page schema** — paste any JSON-LD for one URL (e.g. `Event`, `HowTo`, `VideoObject`, `Service`), either added to the auto schema or replacing it.

Admin features: JSON validation on save, a live preview of the final JSON-LD for the URL, a "Test in Rich Results" button (opens Google's test with the URL), and an audit flag when required properties are missing.

### Auto schema by page type

| Page | Types |
|---|---|
| All pages | `WebSite` (with `SearchAction` on home), `Organization` (logo, contact points incl. toll-free, `sameAs` socials, addresses) |
| Home | + `FAQPage` if there's an FAQ block |
| Product — Buy | `Product` with `brand`, `sku`, `image`, `description`, `additionalProperty` (capacity, layers…), `offers` (`Offer` / `AggregateOffer`, INR price, availability, `shippingDetails`, `hasMerchantReturnPolicy`); `BreadcrumbList`; `FAQPage` |
| Product — Enquiry | `Product` **without** `offers` (no fake price); `BreadcrumbList`; `FAQPage` |
| Collection | `CollectionPage` + `ItemList` of products; `BreadcrumbList` |
| Blog post | `BlogPosting` (author `Person` with profile URL, `datePublished`, `dateModified`, image); `BreadcrumbList` |
| Author page | `ProfilePage` + `Person` |
| Job | `JobPosting` (title, description, `datePosted`, `validThrough`, `employmentType`, `hiringOrganization`, `jobLocation`, `baseSalary` if shown). Removed when the job closes |
| Contact | `ContactPage` + Organization contact points |
| About | `AboutPage` |
| Dealer locator / city pages | `LocalBusiness` per dealer (phase 3 city pages) |
| Video blocks | `VideoObject` |

**Important:** Shopify product cards currently show "Rs. 1.00". That placeholder must never reach `offers.price`, or Google will show ₹1 in search results. Enquiry products have no `offers` at all.

## 3. Technical SEO built in

- Server-rendered HTML for all public content (Next.js); animations never hide content from crawlers.
- `<title>`, meta, canonical and OG tags come from `generateMetadata`, fed by the API's `seo` object.
- Clean HTML: semantic landmarks, one H1, logical heading order from block templates.
- XML sitemap index: `pages`, `collections`, `products` (with image entries), `posts`, `jobs`. `lastmod` comes from `updated_at`; only indexable, published URLs are listed.
- robots.txt is managed in admin; staging always sends `Disallow: /` + `X-Robots-Tag: noindex`.
- Pagination: `?page=n` is self-canonical. Sort/filter parameters canonicalise to the clean URL, and filter combinations are `noindex`. The SEO team can turn valuable filters into indexable landing pages (e.g. "1000 litre water tank").
- No trailing slashes; lowercase URLs enforced; `vectus.in` → `www.vectus.in`.
- Images: next/image (AVIF/WebP, responsive sizes, lazy below the fold), alt text from the media library, descriptive file names on upload.
- Performance budget: UI_ANIMATION.md §6.
- Internal links: breadcrumbs, related products, "products in this post", footer category links, collection descriptions.
- Hreflang: not needed while the site is English-only; ready if Hindi is added (`/hi/...`).
- Google Merchant Center feed (`/feeds/google-merchant.xml`) for Buy-mode products.

## 4. Migration checklist

**Before build (Phase 0)**
- [ ] Full crawl of the current site (Screaming Frog): every URL with status, title, description, H1, canonical, word count, images
- [ ] Shopify export: products (CSV + API for metafields), collections, pages, blog posts, existing Shopify URL redirects, files
- [ ] Search Console: top pages & queries (16 months), backlinks to top pages, index coverage
- [ ] GA4: landing-page traffic baseline per URL
- [ ] Record existing schema and Shopify-added meta

**Before launch**
- [ ] Every crawled URL mapped in PAGE_INVENTORY.md (Keep / 301 / 410)
- [ ] Redirects imported; no chains (A→B→C) or loops
- [ ] Crawl staging (with auth) and diff against the live crawl: titles, descriptions, H1s, canonicals, status codes
- [ ] Content parity on the top 100 URLs (no lost text, FAQs or images)
- [ ] Schema validated for each template
- [ ] Sitemaps generated; production robots.txt ready
- [ ] GA4/GTM/GSC tags in place; e-commerce events tested
- [ ] Environment check so staging `noindex` cannot leak to production

**Launch day**
- [ ] DNS switch; HTTPS valid on www and apex
- [ ] Crawl production immediately; fix any 4xx/5xx
- [ ] Submit the sitemap in GSC; request indexing for top pages
- [ ] Old Shopify sitemap URLs redirect to the new sitemap

**After launch (4–8 weeks)**
- [ ] Daily: GSC coverage, 404 log → redirects
- [ ] Weekly: rankings & traffic for top URLs vs baseline; Core Web Vitals report
- [ ] Keep the Shopify store paused (not deleted) for 30 days for reference
