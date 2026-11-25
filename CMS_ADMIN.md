# Admin Panel (Filament) — admin.vectus.in

One admin panel with different screens per role. Everything shown on the public site can be edited here.

## 1. Navigation

| Group | Screens |
|---|---|
| Dashboard | Role-specific widgets: today's leads, orders, open claims, SEO warnings, new applications |
| Content | Pages · Menus · Media library · Galleries · Downloads · Brands · Client logos · Testimonials |
| Catalog | Products · Collections · Attribute sets · Attributes · (Reviews) |
| Sales | Orders · Customers · Coupons · Shipments · Returns/Refunds · Abandoned carts |
| SEO | SEO overview & bulk editor · Meta templates · Schema templates · Redirects · 404 log · Sitemap & robots · Tracking scripts · SEO audit |
| Blog | Posts · Authors · Tags |
| Careers | Job openings · Applications (Kanban + table) · Departments |
| Leads | Enquiries · Dealer applications · Newsletter · Routing rules |
| Dealers | Dealers (map view, CSV import) |
| Warranty (R2) | QR batches · Product units · Registrations · Claims (board) · Policies · Plants · Reports |
| Settings | General · Contact & offices · Social · Floating buttons · Commerce (store on/off, shipping, tax) · Integrations (keys) · Notification templates |
| System | Staff users · Roles & permissions · Activity log · Content revisions |

## 2. Roles & permissions

✔ = full access, R = read-only, — = no access

| Module | Super Admin | SEO Manager | Content Editor | HR | Sales / Leads | Store Manager | Service Manager | Service Agent |
|---|---|---|---|---|---|---|---|---|
| Pages, menus, media | ✔ | ✔ | ✔ (draft + submit) | — | — | — | — | — |
| Products & collections | ✔ | ✔ (content + SEO) | ✔ (content) | — | R | ✔ (price, stock, Buy/Enquiry) | R | — |
| SEO module (meta, schema, redirects, scripts) | ✔ | ✔ | — | — | — | — | — | — |
| Blog | ✔ | ✔ | ✔ (draft + submit) | — | — | — | — | — |
| Careers & applications | ✔ | Job page SEO only | — | ✔ | — | — | — | — |
| Leads & dealer applications | ✔ | — | — | — | ✔ | R | — | — |
| Dealers | ✔ | R | — | — | ✔ | — | R | R |
| Orders, customers, coupons | ✔ | — | — | — | R | ✔ | R (customers) | — |
| Warranty registrations & claims | ✔ | — | — | — | — | — | ✔ | ✔ (own region / assigned only) |
| QR batches | ✔ | — | — | — | — | — | ✔ | — |
| Settings, users, roles | ✔ | Tracking scripts only | — | — | — | Commerce settings | — | — |

Roles and permissions are editable in admin, so the client can create new combinations later. Customer PII (mobile, address) is masked for roles that don't need it.

## 3. Page builder

A page is a list of **blocks**. Editors add, drag to reorder, duplicate and hide blocks per device, and can set an anchor ID. Each block has three tabs: **Content**, **Design** (theme: light, dark or brand; spacing) and **Animation** (a preset from UI_ANIMATION.md §4, delay, on/off).

| Block | Fields | Notes |
|---|---|---|
| Hero — video | Heading, subheading, video, poster image, CTAs | The poster is the LCP image; video lazy-plays |
| Hero — slider | Slides (desktop/mobile image, heading, CTA) | |
| Category tiles | Title, tiles (image, label, link) | |
| Product grid / carousel | Source: collection, manual or featured; limit; layout | Cards include Compare + Buy/Enquire |
| Product showcase (assembly) | Product, animation type, copy per step | Scroll-driven assembly |
| Layer explainer | Layers (name, description, colour/image) | Animated stacked layers (e.g. Ten-X's 10 layers) |
| Rich text | WYSIWYG (H2–H4, lists, tables, links, images) | H1 not allowed — the page template owns the one H1 |
| Image + text | Image, text, side, CTA | |
| Stats counter | Items (number, suffix, label) | Counts up on scroll |
| Timeline | Items (year, title, text, image) | "Since 1989" story |
| Team grid | People (photo, name, role, bio) | Board / management |
| Logo wall | Source: client logos or brands | Marquee |
| Plant map | Locations (from Plants) | India map with pins |
| Latest posts | Blog, tag, count | |
| FAQ | Q&A items | Emits FAQPage schema |
| Enquiry form | Form type, heading, default query type | |
| CTA banner | Heading, text, button, background | |
| Video | YouTube / Vimeo / upload | Thumbnail facade until clicked (performance) |
| Gallery | Gallery source, layout | Lightbox |
| Downloads list | Category / product filter | |
| Dealer locator | Default state | |
| Instagram feed | Count | |
| Compare | Fixed set of products | For SEO comparison pages ("Ten-X vs Granito") |
| Tabs / accordion | Items with rich text | |
| Table | Rows / columns | e.g. GST info |
| Custom HTML | Code | Super Admin only |

## 4. Editorial workflow

- **Draft → In review → Published → Archived.** Content Editors can only submit for review; the SEO Manager or Super Admin publishes.
- **Preview** any draft on the real frontend (Next.js draft mode).
- **Schedule** publish and unpublish dates.
- **Revisions:** every save stores a snapshot that can be compared and restored.
- **Slug change** warns the editor and auto-creates a 301 from the old URL.
- **Deleting** a published page or product asks for a redirect target (or 410).
- **Activity log** shows who changed what.

## 5. Product editor

Tabs:
- **General:** name, brand, tagline, descriptions, badges
- **Variants & pricing:** options, SKU, MRP, price, stock, dimensions/weight
- **Purchase:** Buy or Enquiry per product, with per-variant override
- **Specifications:** attribute-set fields, with filterable/comparable flags
- **Media:** images with alt text, 360°, assembly frames / 3D model, video, brochures
- **Collections:** primary + others
- **Related:** related products, accessories, upsells
- **FAQs**
- **Warranty:** policy, years
- **SEO:** see SEO.md
- **Revisions**

Bulk actions: change purchase mode, assign collection, publish/unpublish, CSV export/import (including prices and SEO fields).

## 6. Blog

- Rich editor with headings, images (alt text required), tables, embeds, callouts, product cards, CTA blocks and FAQ blocks.
- Fields: blog (News or Publications), title, slug, excerpt, featured image, author, tags, related products, publish date, reading time (automatic).
- Author profiles (photo, designation, bio, LinkedIn) for E-E-A-T.
- SEO tab, scheduling, preview, revisions, internal-link suggestions (posts and products that share tags or keywords).

## 7. Careers (HR)

- **Job openings:** title, department, location, type, experience, salary range (show/hide), number of openings, description, responsibilities, requirements, benefits, valid-through date, status. A job auto-closes after its valid-through date: the page shows "position closed" (or 301s, HR's choice) and its JobPosting schema is removed.
- **Applications:** table and Kanban by status (New → Screening → Shortlisted → Interview → Offered → Hired / Rejected / On hold). In-browser CV preview, notes, rating, status history, bulk status change, CSV export.
- **Emails:** automatic acknowledgement to the candidate; notification to HR (and the hiring manager) per job; optional templated rejection.
- **General applications** when no job matches.
- **Retention:** applications are deleted automatically after 12 months unless the candidate agreed to longer.
- HR users see only the Careers section.

## 8. Leads & enquiries

- Every site form lands in **Enquiries** with its type, product, source page and UTM data.
- **Routing rules** by state and lead type assign staff and set notification emails (e.g. Rajasthan leads → Jaipur team).
- Status pipeline, notes, duplicate detection (same phone within 30 days), spam flag, CSV export, optional CRM push via webhook later.
- **Dealer applications:** the online form replaces the PDF, with a review workflow and uploaded documents.

## 9. Dealers

CSV import (name, type, address, pincode, phone; lat/lng optional — missing ones are geocoded), map view, active toggle. Dealers power `/pages/dealer-locator`, the dealer picker in warranty registration, and future city pages.

## 10. Settings

- **General:** logo, favicon, site name
- **Contact:** toll-free 1800-202-6666, WhatsApp, emails, registered & corporate office addresses, map pins
- **Social links**
- **Floating buttons:** enquiry / call / WhatsApp on/off and numbers
- **Commerce:** store on/off, compare on/off, free-shipping threshold, COD on/off, invoice prefix, GSTIN
- **Notification templates:** SMS DLT template IDs, WhatsApp templates, emails
- **Integrations:** API keys, stored encrypted
