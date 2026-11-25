# Architecture

## 1. Overview

```
                         Cloudflare  (DNS · CDN · WAF · Turnstile)
          ┌───────────────────────┼───────────────────────────┐
          ▼                       ▼                           ▼
   www.vectus.in           api.vectus.in               admin.vectus.in
   Next.js "web"           Laravel API      ── same ──  Filament admin
   (SSR/ISR, BFF routes)   /api/v1/*         Laravel app (staff only)
          │  ▲                    │
  JSON    │  │ revalidate         ├── MySQL 8 ............ all data
  (cached)│  │ webhook (tags)     ├── Redis .............. cache, queues, rate limits
          ▼  │                    ├── Object storage ..... media → media.vectus.in (CDN)
        Laravel ──────────────────┤
                                  ├── Meilisearch ........ product / blog search
                                  └── Queue workers ...... SMS, WhatsApp, email, PDFs,
                                                           Shiprocket, image processing

   External: Razorpay · Shiprocket · MSG91 (SMS OTP) · WhatsApp BSP · Email (SES / ZeptoMail)
             Google Maps · GA4 / GTM · Search Console · Instagram Graph API
```

- **Next.js** renders every public page on the server (good for SEO), fetches content from the Laravel API, and stores no customer data of its own.
- **Laravel** is the single source of truth: content, catalog, orders, customers, warranty. Filament runs inside the same Laravel app.
- **Staff** use admin.vectus.in. **Customers** use www.vectus.in (account, cart, warranty).

## 2. Stack

| Concern | Choice | Notes |
|---|---|---|
| Frontend framework | Next.js 16+ (App Router), React 19, TypeScript | Server Components, ISR, `generateMetadata`, draft mode for previews |
| Styling | Tailwind CSS v4 + CSS variables for design tokens | |
| Animation | GSAP 3.13+ (ScrollTrigger, SplitText, Flip) + Lenis | All GSAP plugins are free since 3.13. See UI_ANIMATION.md |
| 3D (optional) | React Three Fiber + drei, GLB/Draco models | Only for interactive 360° viewers |
| Forms | React Hook Form + Zod | The same Zod schemas validate in server routes |
| Client state | Zustand (cart, compare tray) | Persisted to localStorage where needed |
| Backend | PHP 8.3+, Laravel 12+ | |
| Admin panel | Filament 4+ | CRUD, tables, filters, roles, dashboards |
| Auth & roles | Laravel Sanctum (customers), Filament auth + spatie/laravel-permission (staff) | |
| Media | spatie/laravel-medialibrary → S3/R2; responsive WebP/AVIF conversions | |
| Audit | spatie/laravel-activitylog | Who changed what, and when |
| Search | Laravel Scout + Meilisearch | Typo-tolerant product & blog search, facets |
| PDFs | Browsershot or DomPDF | Invoices, warranty certificates |
| QR | endroid/qr-code or chillerlan/php-qrcode | Batch label generation (R2) |
| Database | MySQL 8.4 LTS, InnoDB, utf8mb4 | |
| Cache / queue | Redis + Laravel Horizon | |
| Monitoring | Sentry (both apps), uptime monitor, Laravel Pulse | |
| CI/CD | GitHub Actions | Lint, type-check, tests, deploy |

## 3. Why Laravel instead of plain PHP

The client asked for PHP + MySQL. Laravel is PHP + MySQL, and it saves months of custom work:

- Filament gives a production-grade admin panel (CRUD, roles, filters, dashboards) for every module, so the SEO, HR, sales and service teams each get their own screens.
- Queues, mail, validation, migrations, rate limiting, encryption and CSRF protection come built in.
- Mature packages exist for permissions, media, audit logs and payments.
- Handover is easy: most PHP developers in India know Laravel.

If the client insists on core PHP, add roughly 6–8 weeks just for the admin panel and security plumbing.

## 4. Rendering & caching

| Route type | Rendering | Revalidation |
|---|---|---|
| Home, CMS pages, collections, products, blog, careers | Server-rendered and cached (ISR) | On demand via cache tags when content is saved in admin; time-based fallback (e.g. 1 h) |
| Price / stock / Buy availability on product pages | Fetched fresh on the client after load (small JSON call) | None — always live |
| Search, compare, cart, checkout, account, QR scan | Dynamic, not cached | — |
| Sitemaps, robots.txt | Generated from the API, cached | Revalidated on publish |

**Publish flow**

1. An editor saves a product, page or post in Filament.
2. A Laravel model observer dispatches a queued job → `POST https://www.vectus.in/api/revalidate` with a shared secret and tags, e.g. `["product:vectus-ten-x-water-tank", "collection:water-tanks", "sitemap"]`.
3. Next.js calls `revalidateTag()` for each tag. The change is live within seconds.

Tag names: `page:{slug}`, `product:{slug}`, `collection:{slug}`, `post:{slug}`, `job:{slug}`, `menu:{location}`, `settings`, `redirects`, `sitemap`.

**Preview:** "Preview" in Filament opens `https://www.vectus.in/api/preview?token=…&path=…`, which turns on Next.js draft mode and fetches unpublished content.

## 5. Authentication

| User | Method | Details |
|---|---|---|
| Staff | Email + password + 2FA (TOTP) in Filament | Roles via spatie/laravel-permission. Optional IP allowlist for admin.vectus.in |
| Customers | Mobile number + OTP (SMS, WhatsApp fallback) | One account for orders **and** warranties. Email optional |

**Customer session (BFF pattern).** The browser never holds the API token.

1. Browser → Next.js route handler `/api/auth/verify-otp`.
2. Next.js → Laravel `POST /api/v1/auth/otp/verify` → receives a Sanctum token.
3. Next.js stores the token in an `httpOnly`, `Secure`, `SameSite=Lax` cookie and proxies authenticated calls (`/api/account/*`, `/api/cart/*`, `/api/checkout/*`) to Laravel with the token attached.

## 6. Repository layout

```
vectus/
├── web/                                Next.js
│   ├── app/
│   │   ├── (site)/
│   │   │   ├── page.tsx                      home (CMS page "home")
│   │   │   ├── pages/[slug]/page.tsx         CMS pages (keeps Shopify URLs)
│   │   │   ├── collections/[slug]/page.tsx   product listing
│   │   │   ├── products/[slug]/page.tsx      product detail
│   │   │   ├── blogs/[blog]/page.tsx         blog listing (news, publications)
│   │   │   ├── blogs/[blog]/[slug]/page.tsx  blog post
│   │   │   ├── careers/…                     job listing, detail, apply
│   │   │   ├── compare/page.tsx
│   │   │   ├── search/page.tsx
│   │   │   ├── cart/ · checkout/ · order/[orderNo]/
│   │   │   ├── account/…                     orders, addresses, products, warranties, claims
│   │   │   ├── w/[code]/page.tsx             QR scan resolver (R2)
│   │   │   └── policies/[slug]/page.tsx
│   │   ├── api/                              revalidate, preview, BFF proxies
│   │   ├── sitemap.xml/route.ts · robots.ts
│   │   └── not-found.tsx
│   ├── components/
│   │   ├── blocks/                           one component per CMS block type
│   │   ├── motion/                           Lenis provider, GSAP setup, AssemblySequence
│   │   ├── commerce/                         buy box, cart drawer, compare tray
│   │   └── ui/                               design-system primitives
│   ├── lib/  api/ · seo/ (metadata, JSON-LD) · utils/
│   └── proxy.ts                              redirects + legacy Shopify URL handling
│                                             (called middleware.ts before Next.js 16)
└── api/                                Laravel
    ├── app/
    │   ├── Models/
    │   ├── Http/Controllers/Api/V1/
    │   ├── Http/Resources/                   JSON shapes for the frontend
    │   ├── Filament/Resources/               admin screens per module
    │   ├── Services/                         Payments, Shipping, Sms, WhatsApp, Qr, Schema, Revalidate
    │   └── Jobs/ · Observers/ · Policies/
    ├── database/migrations/ · seeders/
    └── routes/api.php
```

## 7. Environments & deployment

| Env | Web | API / Admin | Data |
|---|---|---|---|
| Local | `next dev` | Laravel Sail / Herd | Local MySQL |
| Staging | staging.vectus.in (password-protected, `noindex`) | api-staging / admin-staging | Copy of production with PII masked |
| Production | www.vectus.in | api.vectus.in / admin.vectus.in | Managed MySQL, daily backups + point-in-time recovery |

**Hosting (recommended: India region, for latency and data location)**

- Laravel: AWS EC2/Lightsail in Mumbai (ap-south-1) or DigitalOcean Bangalore, behind Cloudflare; managed MySQL (RDS / DO Managed DB); Redis.
- Next.js: Vercel (Mumbai function region) **or** self-hosted Node (Docker/PM2) on the same cloud. Either way Next.js stores no customer data.
- Media: S3 + CloudFront or Cloudflare R2 at `media.vectus.in`.

Deploys run through GitHub Actions: `main` → staging automatically, a tagged release → production. Migrations run on deploy, with zero downtime (Deployer/Envoy or containers).

## 8. Domains & DNS cutover

- Move DNS to Cloudflare **before** launch and copy every existing record. In particular keep the email MX/SPF/DKIM/DMARC records, `vectusconnect.vectus.in` (employee login) and any other subdomains in use.
- Replace the Shopify domain records only at cutover.
- `vectus.in` → 301 → `https://www.vectus.in` (keeps the current canonical host).

## 9. Security & compliance

- HTTPS everywhere, HSTS, security headers (CSP, X-Frame-Options, Referrer-Policy).
- Cloudflare WAF + Turnstile on every public form; Laravel rate limits on OTP, forms and QR lookups.
- Card data never touches our servers (Razorpay Checkout). Webhook signatures are verified.
- CVs, invoices and claim photos go in **private** storage and are served only through short-lived signed URLs to authorised staff or the owner.
- Admin: 2FA mandatory, least-privilege roles, activity log on every model, session timeout.
- Customer data under India's DPDP Act 2023: explicit consent checkboxes, purpose-specific notices, an updated privacy policy, data export/delete on request, retention rules (e.g. job applications deleted after 12 months unless the candidate consents to longer).
- Backups: daily database, weekly media; restore tested before launch.
- Monthly dependency updates; Sentry for errors.

## 10. Integrations

| Service | Purpose | Release |
|---|---|---|
| Razorpay | Payments (UPI, cards, netbanking, wallets), refunds, webhooks | R1 |
| Shiprocket | Pincode serviceability, shipping rates, AWB, tracking webhooks | R1 |
| MSG91 (or another DLT-registered provider) | OTP + transactional SMS | R1 |
| WhatsApp Business API (via a BSP) | Order / warranty / claim updates | R1 optional, R2 |
| Amazon SES or Zoho ZeptoMail | Transactional email | R1 |
| Google Maps Platform | Dealer locator map + geocoding | R1 |
| Meilisearch | Search | R1 |
| GA4 + GTM, Search Console, Meta Pixel | Analytics, e-commerce events | R1 |
| Instagram Graph API | "#Vectus" feed block | R1 |
| Google Merchant Center | Free listings / Shopping feed for buyable products | R1 |
| Client ERP / CRM | Stock, serials, leads — TBD | TBD |

## 11. Conventions

- API: versioned `/api/v1`, JSON, snake_case keys, a Laravel API Resource for every response shape.
- TypeScript types for API responses live in `web/lib/api/types.ts` (generated from OpenAPI once stable).
- Every public entity has a `slug`, a `status` (`draft`, `published`, `archived`), `published_at`, and SEO meta.
- Slugs are unique per entity type; changing a slug auto-creates a 301.
- Money is stored as integer paise (`price_paise`) and displayed in INR.
- Dates are stored in UTC and displayed in IST.
