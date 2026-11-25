# CLAUDE.md

Guidance for AI coding assistants working in this repository.

## Project

Rebuild of www.vectus.in (currently Shopify) as:

- `web/` — Next.js App Router, TypeScript, Tailwind, GSAP + Lenis
- `api/` — Laravel REST API (`/api/v1`) + Filament admin, MySQL 8, Redis

The planning docs are in the repo root. Read the relevant one before working on a module:
README · PROJECT_PLAN · ARCHITECTURE · PAGE_INVENTORY · DATABASE_SCHEMA · API_SPEC · CMS_ADMIN · SEO · ECOMMERCE · WARRANTY_PORTAL · UI_ANIMATION · OPEN_QUESTIONS

## Rules that must not be broken

- **URLs:** keep the Shopify patterns (`/products/{slug}`, `/collections/{slug}`, `/pages/{slug}`, `/blogs/{blog}/{slug}`, `/policies/{slug}`). Any URL change needs a 301 entry.
- **SEO data comes from the API.** Pages render `seo.title`, `seo.description`, `seo.canonical`, `seo.robots`, `seo.og` and `seo.json_ld` from the API. No hardcoded meta in components.
- **One H1 per page**, owned by the page template; CMS rich text cannot add an H1.
- **Content is server-rendered.** Never hide content with CSS `opacity: 0` before JS runs; reveal styles apply only under the `.js-motion` class.
- **Animations:** use `useGSAP` with a scope; animate only transform/opacity; respect `prefers-reduced-motion`; never delay the LCP element.
- **Purchase mode:** a variant is purchasable only if its effective mode is `buy`, the store is enabled, it has a price, it's in stock and the pincode is serviceable. Enquiry products never show a price and never output `offers` in schema.
- **Money** is integer paise; **dates** are UTC in the database and IST in the UI.
- **The customer API token** lives only in an httpOnly cookie set by Next.js route handlers. Never expose it to client JS.
- **Private files** (CVs, invoices, claim photos) go on the private disk and are served via signed URLs.
- **Revalidation:** every change to a publishable model must dispatch revalidation tags (`product:{slug}`, `collection:{slug}`, `page:{slug}`, `post:{slug}`, `job:{slug}`, `menu:{location}`, `settings`, `redirects`, `sitemap`).

## Status

Planning stage — no application code yet. Open client questions are in OPEN_QUESTIONS.md.
