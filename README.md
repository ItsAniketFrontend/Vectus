# Vectus.in — Website Rebuild

Rebuild of [www.vectus.in](https://www.vectus.in/) (Vectus Polymers Pvt. Ltd.: water storage tanks, bathware, household plastics, gardening and road-safety products) from Shopify to a custom stack.

| Layer | Choice |
|---|---|
| Frontend | Next.js (App Router, TypeScript, Tailwind), animated with GSAP + Lenis |
| Backend | PHP — Laravel REST API + Filament admin panel |
| Database | MySQL 8 |
| Cache / queues | Redis |
| Media | S3-compatible object storage behind a CDN |

## What the new site must do

1. Carry over **every page** of the current Shopify site without losing search rankings.
2. Let the **SEO team** manage content, meta titles/descriptions, on-page elements, schema markup and redirects without a developer.
3. **Sell some products online** (Add to Cart / Buy Now); every other product shows an **enquiry form**. Set per product.
4. **Add to Compare** for products.
5. **Blog** posted from the admin panel.
6. **Careers** page managed by HR, with online applications.
7. **Customer portal**: scan the QR code on a product to register its warranty, download the certificate and raise service claims.
8. **Modern UI/UX**: smooth scrolling, scroll-driven animations, product assembly animations.

## Documents

| File | What it covers |
|---|---|
| [PROJECT_PLAN.md](PROJECT_PLAN.md) | Scope, releases, phases, timeline, team, risks, launch criteria |
| [ARCHITECTURE.md](ARCHITECTURE.md) | System design, stack, caching, auth, hosting, security, repo layout |
| [PAGE_INVENTORY.md](PAGE_INVENTORY.md) | Every current page/collection mapped to the new site + redirect plan |
| [DATABASE_SCHEMA.md](DATABASE_SCHEMA.md) | MySQL tables |
| [API_SPEC.md](API_SPEC.md) | REST endpoints between Next.js and Laravel |
| [CMS_ADMIN.md](CMS_ADMIN.md) | Admin panel modules, roles, page builder, blog, careers, leads |
| [SEO.md](SEO.md) | SEO controls, dynamic schema, sitemaps, migration checklist |
| [ECOMMERCE.md](ECOMMERCE.md) | Buy vs Enquiry, cart, checkout, payments, shipping, compare |
| [WARRANTY_PORTAL.md](WARRANTY_PORTAL.md) | QR codes, warranty registration, claims, customer portal |
| [UI_ANIMATION.md](UI_ANIMATION.md) | Design direction, GSAP + Lenis setup, assembly animations, performance rules |
| [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) | Questions the client must answer, decision log |
| [CLAUDE.md](CLAUDE.md) | Working rules for AI-assisted development |

## Planned repository layout

```
vectus/
├── web/          Next.js frontend (www.vectus.in)
├── api/          Laravel API + Filament admin (api.vectus.in, admin.vectus.in)
└── *.md          Planning docs (this folder)
```

## Status

Planning (started 2026-11-25). Build is blocked on the items marked **Blocker** in [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).
