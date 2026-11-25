# Page Inventory & URL Map

Goal: **every page on the current Shopify site exists on the new site** at the same URL, or 301-redirects to its replacement.

Source: a crawl of www.vectus.in on 2026-11-25 (homepage, sitemaps, contact, about, warranty and dealer-locator pages). Product pages and `/collections/all` were rate-limited during that crawl, so product counts are estimates. **Phase 0 must redo this with a full crawl (Screaming Frog), the Shopify admin export and Search Console data.** After that, this file becomes the signed-off redirect map.

## 1. URL strategy

Keep Shopify's URL patterns at launch:

| Pattern | Keep? |
|---|---|
| `/products/{slug}` | Yes, unchanged |
| `/collections/{slug}` | Yes, unchanged |
| `/pages/{slug}` | Yes, unchanged |
| `/blogs/news/{slug}`, `/blogs/publications/{slug}` | Yes, unchanged |
| `/policies/{slug}` | Yes, unchanged |

Changing platform and URLs at the same time is the most common cause of ranking loss. Only the cleanups listed below change URLs. If the SEO team wants a new structure (e.g. `/water-tanks/{slug}`), do it later as a separate project.

**Action** legend: **Keep** = same URL, rebuilt · **301** = redirect to target · **Review** = SEO team decides in Phase 0 · **New** = new URL.

## 2. CMS pages (`/pages/*`)

| Current URL | Content | New implementation | Action |
|---|---|---|---|
| `/` | Homepage | CMS page `home` (block builder) | Keep |
| `/pages/about-us` | About, stats, board, management, clients, CSR | CMS page; sections keep the anchors `#board-of-directors`, `#management-team`, `#our-clients`, `#csr` | Keep |
| `/pages/mission-and-vision` | Mission & vision | CMS page | Keep |
| `/pages/gst-registration` | GST info | CMS page (table block) | Keep |
| `/pages/contact` | Contact details, enquiry form, FAQ | CMS page + Enquiry Form block + Map block. **Remove the leftover Shopify placeholder FAQ text** | Keep |
| `/pages/career` | Careers | Careers module listing | **301 → `/careers`** |
| `/pages/gallery` | Image gallery | Gallery module | Keep |
| `/pages/articles-publications` | Publications | CMS page listing the `publications` blog | Keep |
| `/pages/publication` | Older publications page | — | **301 → `/pages/articles-publications`** |
| `/pages/downloads` | Downloads (brochures etc.) | Downloads module | Keep |
| `/pages/dealer-locator` | Dealer locator (currently only the enquiry form) | Dealer Locator module (map + search by state/city/pincode) | Keep |
| `/pages/find-a-dealer` | Duplicate locator | — | **301 → `/pages/dealer-locator`** |
| `/pages/store-locator` | Duplicate locator | — | **301 → `/pages/dealer-locator`** |
| `/pages/warrenty-registration` | Warranty registration (typo in slug; currently only the enquiry form) | — | **301 → `/pages/warranty-registration`** |
| — | — | Warranty registration landing (R1: manual form; R2: scan/enter code + portal) | **New `/pages/warranty-registration`** |
| `/pages/comparison-products-page` | Product comparison | Compare feature | **301 → `/compare`** |
| `/pages/water-storage-tank` | Category landing | CMS page with blocks + product grid | Keep |
| `/pages/household-plastic` | Category landing | CMS page | Keep |
| `/pages/bathware-range` | Category landing | CMS page | Keep |
| `/pages/gardening-products` | Category landing | CMS page | Keep |
| `/pages/road-safety-products` | Category landing | CMS page | Keep |
| `/pages/ptmt-taps` | Sub-category landing | CMS page | Keep |
| `/pages/kitchen-sink` | Sub-category landing | CMS page | Keep |
| `/pages/shower-and-health-faucets` | Sub-category landing | CMS page | Keep |
| `/pages/cistern-tanks` | Sub-category landing | CMS page | Keep |
| `/pages/toilet-seat-covers` | Sub-category landing | CMS page | Keep |

> **Review:** several category landing pages (e.g. `/pages/ptmt-taps`) overlap with collections (`/collections/ptmt-taps`). Two URLs aimed at the same keyword compete with each other. In Phase 0 the SEO team picks one canonical per topic. Usually either the landing page targets the head term and the collection handles listings, or the landing content moves onto the collection page and the other URL 301s.

## 3. Policy pages

| Current URL | Action |
|---|---|
| `/policies/privacy-policy` | Keep (update for the DPDP Act and new data flows) |
| `/policies/terms-of-service` | Keep (update for online sales) |
| `/policies/refund-policy` | **New** — required by Razorpay and Google Merchant Center |
| `/policies/shipping-policy` | **New** — required for online sales |
| `/policies/warranty-policy` | **New** — warranty terms by product line |

## 4. Collections (`/collections/*`)

All rebuilt at the same URL as `collections` records (type `category`, `series`, `curated` or `brand`). Product counts per collection get filled in during Phase 0.

**Top level / product types**

| Slug | Type | Action |
|---|---|---|
| `water-tanks` | category | Keep |
| `overhead-tanks`, `loft-tanks`, `underground-tanks` | category (child of water-tanks) | Keep |
| `bathware` | category | Keep |
| `bathware-1` | ? | Review — likely a duplicate of `bathware` → 301 |
| `household-plastic` | category | Keep |
| `gardening-products` | category | Keep |
| `gardening-products-1` (titled "Plastic products") | ? | Review — rename or 301 |
| `maali` | brand | Keep |
| `road-safety-products` | category | Keep |
| `best-of-vectus` | curated | Keep |
| `all-products` | curated (all) | Keep |
| `all` | Shopify built-in | Keep as "all products", or 301 → `all-products` |
| `frontpage` | Shopify default | 301 → `/` (or noindex) |

**Tank series:** `smart-series`, `smart-dual-series`, `vectus-pro-cistern-tank`, `vectus-centra-flush-tank`, `vectus-smart-flush-tank`, `vectus-chic` — Keep.

**Household / garden / other:** `indus-series`, `patio-series`, `europa-series`, `ridges-series`, `classico-series`, `garden-pipes`, `agri-products`, `kitchen-products`, `plastic-bathroom-accessories`, `roadside-bins`, `community-bins`, `household-bins`, `pe-manhole` — Keep.

**Road safety:** `barricades-traffic-cones`, `safety-accessories` — Keep.

**Bathware & taps:** `ptmt-taps`, `ptmt-bath-accessories`, `cistern-tanks`, `toilet-seat-covers`, `vectus-smart-toilet-seat-covers`, `vectus-silk-toilet-seat-covers`, `vectus-pro-toilet-seat-cover`, `kitchen-sink`, `kitchen-sink-silk-series`, `vectus-kitchen-sink-silk-series`, `shower-and-health-faucets`, `vectus-shower`, `health-faucets`, `sonic-series-taps`, `presto-series`, `maria-single-lever-basin-mixer`, `silk-series`, `wave-series`, `rhino-series`, `casa-series`, `tee-series`, `renova-series`, `chic-series`, `chic-dual-series`, `eco-series`, `pro-series` — Keep.
Review: `kitchen-sink-silk-series` and `vectus-kitchen-sink-silk-series` look like duplicates.

## 5. Products (`/products/*`)

- Roughly 400 product URLs (estimated from sitemap size; the exact count comes from the Shopify export).
- All Keep at `/products/{slug}`.
- Import from Shopify: title, slug, description HTML, images + alt text, variants/options, tags, collections, SEO title/description, metafields (layers, UV layers, capacity, etc.). Store `shopify_id` and `shopify_variant_id` for mapping.
- Current product cards show **Rs. 1.00** (a placeholder). Real prices are needed only for products set to Buy; Enquiry products show no price.
- Discontinued products: 301 to their primary collection (never a soft 404).

Sample slugs to test after import: `vectus-ten-x-water-tank`, `vectus-cool-tank`, `vectus-granito-6-layers`, `vectus-safe`, `vectus-silk`, `vectus-loft`, `vectus-isi-tank`, `vectus-puff-tank`, `vectus-horizontal-tank`, `ganga-horizontal-tank`, `waterwell-loft-tank`, `europa-garden-pipes`, `parking-cones`, `vectus-vertical-underground-tank`.

## 6. Blog

| Current URL | Action |
|---|---|
| `/blogs/news` | Keep — blog listing |
| `/blogs/news/{slug}` | Keep — import all posts (count from `sitemap_articles_1.xml`) |
| `/blogs/news/tagged/{tag}` | Keep as tag archive (noindex by default; the SEO team can enable indexing) |
| `/blogs/publications`, `/blogs/publications/{slug}` | Keep |

## 7. Shopify system URLs (handled in `proxy.ts`)

| Shopify URL | New behaviour |
|---|---|
| `/collections/{c}/products/{p}` | 301 → `/products/{p}` |
| `/products/{p}?variant={shopifyVariantId}` | Same URL; map the old variant ID to the new variant via `shopify_variant_id` |
| `/products/{p}.json`, `.js`, `.atom`, `.oembed` | 410 |
| `/cart`, `/cart/*` | `/cart` |
| `/checkout*`, `/{shopId}/checkouts/*`, `/{shopId}/orders/*` | 301 → `/cart` or `/account/orders` |
| `/account`, `/account/login`, `/customer_authentication/*` | 301 → `/account` or `/account/login` |
| `/search?q=` | `/search?q=` (same) |
| `/sitemap.xml` | New sitemap index. Old child sitemaps (`sitemap_products_1.xml` etc.) 301 → `/sitemap.xml` |
| `/cdn/shop/files/*` (e.g. `Customer-opening-form.pdf`) | Linked/indexed files re-hosted on media.vectus.in and 301'd. The dealer PDF form is replaced by an online form |
| `/collections/{c}?page=n`, `?sort_by=` | Supported (pagination is self-canonical; sort params canonicalise to the base URL) |

## 8. Navigation to rebuild (editable in Admin → Menus)

**Header:** Home · About Us (Mission & Vision, Board of Directors, Management Team, GST Info, Our Clients, CSR & Investors) · Products (Water Storage Tanks, Household Plastic, Bathware, Gardening Products, Road Safety Products) · Media (Blog, Publications, Gallery) · Careers · Contact Us (Help & Support) · Search · Account · **Cart (new)** · **Compare (new, with count)**

**Footer:** About Us links · Our Products links · Quick Links (Publication, Image Gallery, Dealer/Distributor Application **→ online form**, Employee Login → vectusconnect.vectus.in, Downloads, Privacy Policy, Terms, **Refund, Shipping and Warranty policies**) · Social (Facebook, Instagram, LinkedIn, X/Twitter, YouTube) · Newsletter · Contact (toll-free 1800-202-6666, WhatsApp, email, registered & corporate offices)

**Floating buttons:** Enquiry · Call · WhatsApp (kept, each configurable in Settings).

## 9. Homepage sections to rebuild (as CMS blocks)

1. Hero video, "Built to perform. Trusted to last." → **Hero (video) block** with animated headline
2. Shop by Category (4 tiles) → **Category Tiles block**
3. Best of Vectus (Ten-X, Cool, Granito, Safe) → **Product Showcase block** with layer/assembly animation
4. Our Clients logo wall (~30) → **Logo Wall block** (marquee)
5. #Vectus Instagram → **Instagram Feed block**
6. Featured Articles (3) → **Latest Posts block**
7. Our Brands (Vectus, Gangotri, Citizen, Premier, Ganga, Waterwell, Maali) → **Brands Strip block** (currently rendered twice — fix)
8. FAQs (7) → **FAQ block** (automatic FAQPage schema)
9. Enquire Now → **Enquiry Form block**

Suggested additions: **Stats block** (since 1989, 4,000+ dealers, 20,000+ retailers, 11 plants), **Plant map block** (manufacturing locations), **Warranty CTA block** ("Scan your tank's QR").

## 10. Forms to rebuild

| Form | Current | New |
|---|---|---|
| Enquiry (site-wide) | First/last name, email, phone, state, city, query type, message | Same fields + product (auto-filled on product pages), consent checkbox, UTM/source capture, Turnstile |
| Product enquiry | — | Enquiry form prefilled with product + variant; optional quantity and pincode |
| Dealer / distributor application | PDF download | Online multi-step form (business details, GST, territory, document upload) |
| Warranty registration | Generic enquiry form | R1: product, serial/code, purchase date, dealer, invoice upload, OTP. R2: QR flow |
| Job application | — | Careers module form with CV upload |
| Newsletter | Email (10% off first purchase) | Same; coupon issued automatically if the offer stays |

## 11. New URLs

`/compare` · `/cart` · `/checkout` · `/order/{orderNo}` (confirmation) · `/account/*` · `/careers`, `/careers/{slug}` · `/pages/warranty-registration` · `/pages/dealer-application` · `/w/{code}` (R2) · `/policies/refund-policy`, `/policies/shipping-policy`, `/policies/warranty-policy`
