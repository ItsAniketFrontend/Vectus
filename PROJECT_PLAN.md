# Project Plan

## 1. Objective

Replace the Shopify site at www.vectus.in with a custom Next.js + Laravel/MySQL platform that:

- keeps current organic traffic and rankings through the migration,
- gives the marketing, SEO, HR, sales and service teams their own admin tools,
- sells selected products online and captures enquiries for the rest,
- adds a QR-based warranty portal for end customers,
- looks and feels like a premium, modern brand site.

## 2. Scope

### In scope

| Area | Includes |
|---|---|
| Website | All current pages, product catalogue (~400 products, ~65 collections), homepage, about, gallery, publications, downloads, dealer locator, contact, policies |
| CMS | Block-based page builder, menus, media library, global settings, revisions, preview, scheduling |
| SEO | Per-page meta, OG tags, canonicals, robots, dynamic JSON-LD schema, redirects manager, XML sitemaps, robots.txt, tracking scripts, SEO audit dashboard |
| E-commerce | Buy vs Enquiry per product, cart, checkout, Razorpay, GST invoices, Shiprocket shipping, orders, coupons, customer accounts |
| Compare | Compare tray + side-by-side spec comparison (up to 4 products) |
| Blog | Posts, authors, tags, scheduling, related products, Article schema |
| Careers | Job openings managed by HR, applications with CV upload, applicant pipeline, JobPosting schema |
| Leads | Enquiry forms, product enquiries, online dealer/distributor application (replaces the PDF), routing by state |
| Warranty portal (Release 2) | QR code generation, scan → register warranty, OTP login, certificate PDF, claims & service tickets, admin claims board |
| UI/UX | New design system, GSAP + Lenis animation, product assembly animations, responsive, accessible |
| Migration | Shopify data export/import, URL preservation, 301 map, analytics continuity |

### Out of scope (unless added)

- Employee portal (vectusconnect.vectus.in) — stays as is, linked from the footer
- ERP replacement or two-way ERP sync (a one-way import/sync can be scoped once the ERP is known)
- Native mobile apps
- Marketplace integrations (Amazon, Flipkart)
- Hindi / regional language versions (the architecture allows adding them later — see OPEN_QUESTIONS)
- Dealer self-service portal

## 3. Release strategy

Two releases, so the SEO-sensitive platform switch doesn't wait on the warranty system's outside dependencies (QR printing, production line, ERP data).

| Release | Contents | Goes live when |
|---|---|---|
| **R1 — Website & Store** | Everything except the QR warranty portal. `/pages/warranty-registration` gets a simple registration form (manual serial entry) | Shopify is switched off |
| **R2 — Warranty Portal** | QR batches, scan flow, customer warranty dashboard, claims | QR format is agreed and the first labelled batch is printed |

## 4. Phases & timeline (estimate)

Assumes the team in §5 and timely client answers and content. Re-estimate after Discovery.

| # | Phase | Weeks | Key outputs |
|---|---|---|---|
| 0 | Discovery & audit | 1–2 | Answers to OPEN_QUESTIONS, full crawl, GSC/GA4 baseline, Shopify data export, final page inventory, signed-off scope |
| 1 | UX/UI design | 2–5 | Wireframes, design system, hi-fi templates (home, collection, product, compare, cart/checkout, blog, careers, account), motion prototypes of 2–3 signature animations |
| 2 | Backend foundation | 3–6 | Laravel + Filament setup, roles, CMS & page builder, catalog, media, SEO module, content API |
| 3 | Frontend build | 5–11 | Layout, block renderer, all templates, Lenis/GSAP system, collection filters, product page, compare, search |
| 4 | Commerce | 8–11 | Cart, OTP login, checkout, Razorpay, GST invoice, Shiprocket, orders admin, emails/SMS |
| 5 | Modules | 9–12 | Blog, careers, leads, dealer locator, gallery, downloads, publications |
| 6 | Assembly animations | 6–12 | 3D renders / frame sequences for flagship products, integrated into product pages + home |
| 7 | Migration | 10–13 | Products, collections, pages, blog, images imported; redirects loaded; content QA |
| 8 | QA, SEO audit, performance, UAT | 12–14 | Test reports, Lighthouse/CWV results, schema validation, client sign-off |
| 9 | Launch R1 + hypercare | 15–16 | DNS cutover, monitoring, GSC checks, bug fixes |
| 10 | R2 Warranty portal | 14–20 | QR system, portal, claims, admin; pilot batch; launch |

Overall: **R1 ≈ 15–16 weeks, R2 ≈ 5–6 more weeks**, overlapping with R1 hypercare.

## 5. Team

| Role | Allocation |
|---|---|
| Project manager / BA | Part-time throughout |
| UI/UX designer (motion-aware) | Full-time weeks 2–6, then part-time |
| Frontend developer (Next.js) | Full-time |
| Creative frontend developer (GSAP / WebGL) | Full-time weeks 5–14 |
| Backend developer (Laravel) ×1–2 | Full-time |
| 3D artist (contract) | Weeks 5–12 — assembly renders |
| QA engineer | Weeks 8–16 and R2 |
| SEO specialist (client or agency) | Discovery, migration, launch, post-launch |

## 6. Milestones & sign-offs

1. Scope & open questions signed off (end of Discovery)
2. Design system + key templates approved
3. Motion prototype approved (before building all animations)
4. Admin panel walkthrough with the SEO / HR / sales teams
5. Content migration complete + redirect map approved by the SEO team
6. UAT sign-off → launch date fixed
7. R2: QR label format approved by production; pilot batch scanned successfully

## 7. Launch criteria (R1)

- [ ] Every URL from the pre-launch crawl and GSC returns 200, or a single-hop 301 to the most relevant page
- [ ] Titles, meta descriptions, H1s and canonicals equal or improve on Shopify for the top 100 URLs by traffic
- [ ] Schema passes the Rich Results Test: Organization, WebSite, BreadcrumbList, Product, FAQPage, BlogPosting, JobPosting
- [ ] XML sitemaps live and submitted; robots.txt correct; no `noindex` left over from staging
- [ ] Lab metrics (mobile) on home / collection / product: LCP ≤ 2.5 s, CLS ≤ 0.1, TBT ≤ 200 ms; Lighthouse performance ≥ 85
- [ ] `prefers-reduced-motion` turns off smooth scroll and heavy animation; all content readable without JS
- [ ] Real-money test orders: pay, invoice, ship, cancel, refund
- [ ] Enquiry, dealer application and job application forms reach the right inboxes and appear in admin
- [ ] Role permissions verified (e.g. HR cannot see orders or customers)
- [ ] GA4, GTM, Search Console and Meta Pixel firing; e-commerce events tracked
- [ ] Backups, monitoring and error tracking active
- [ ] Existing DNS records (email MX, vectusconnect subdomain) preserved

## 8. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Organic traffic drops after migration | High | Keep Shopify URL patterns, full 301 map, pre/post crawl diff, launch on a low-traffic weekday, check GSC daily for 4 weeks |
| Heavy animation hurts Core Web Vitals / SEO | High | Performance budget, server-rendered content, lazy-loaded animation code, lighter effects on mobile (UI_ANIMATION.md) |
| No CAD/3D models for assembly animations | Medium | Ask early; fall back to 2D layered exploded views, or commission models of 2–3 flagship tanks only |
| QR codes already printed on products in an unknown format | High (R2) | Answer before R2 design; build a resolver for legacy codes |
| Payment gateway approval delay | Medium | Apply during Discovery; refund, shipping and contact policy pages must be live on staging |
| Shipping bulky items (tanks) | Medium | Buy mode only where serviceability and weight slabs work; others stay Enquiry |
| Content not ready (copy, images, specs) | Medium | Migrate existing content first; improve content after launch |
| Scope creep from several client teams | Medium | Change requests go through the PM; R1/R2 split |
| SMS DLT template approval delay (India) | Medium | Register sender ID and OTP templates during Discovery |

## 9. After launch

- 4 weeks of hypercare: daily GSC coverage and 404 checks, redirect fixes, bug fixes
- Monthly SEO + performance report
- Phase 3 candidates: Hindi site, programmatic dealer city pages ("water tank dealers in Jaipur"), product reviews & ratings, dealer portal, ERP stock sync, WhatsApp order updates
