# API Specification (v1)

Laravel serves `https://api.vectus.in/api/v1`. The Next.js app is the only intended client. Browsers reach customer endpoints through Next.js BFF routes (ARCHITECTURE.md §5).

## Conventions

- JSON with `snake_case` keys. Money is sent in paise plus a formatted string: `{ "price_paise": 249900, "price_display": "₹2,499" }`.
- Success: `{ "data": …, "meta": { … } }`. Lists add `meta.pagination` = `{ page, per_page, total, last_page }`.
- Errors: `{ "message": "…", "errors": { "field": ["…"] } }` with status 401, 403, 404, 409, 422, 429 or 500.
- Public content endpoints send `Cache-Control` and are cached in Redis; Next.js caches them again with tags.
- Customer endpoints take `Authorization: Bearer <sanctum token>`, added server-side by Next.js.
- Server-to-server calls from Next.js include `X-Api-Key`, so public endpoints can't be scraped at high rates.
- Rate limits: OTP 3 per 10 min per mobile and 10 per hour per IP; forms 5 per 10 min per IP; QR lookup 30 per min per IP.
- Every SEO-able resource includes an `seo` object. The backend resolves all fallbacks (meta templates, auto schema, custom schema), so the frontend just renders it:

```json
"seo": {
  "title": "Vectus Ten-X Water Tank – 10 Layer UV Protected – Vectus",
  "description": "…",
  "h1": "Vectus Ten-X Water Tank",
  "canonical": "https://www.vectus.in/products/vectus-ten-x-water-tank",
  "robots": { "index": true, "follow": true },
  "og": { "title": "…", "description": "…", "image": "https://media.vectus.in/…" },
  "breadcrumbs": [
    { "name": "Home", "url": "/" },
    { "name": "Water Tanks", "url": "/collections/water-tanks" }
  ],
  "json_ld": [ { "@context": "https://schema.org", "@type": "Product" } ]
}
```

## 1. Global & content

| Method | Path | Purpose |
|---|---|---|
| GET | `/settings` | Site name, logo, contact numbers, addresses, social links, floating buttons, feature flags (`store_enabled`, `compare_enabled`) |
| GET | `/menus/{location}` | Menu tree |
| GET | `/pages/{slug}` | CMS page with `blocks`. Block data is pre-resolved (e.g. a product grid block includes its product cards) |
| GET | `/brands` · `/client-logos` | Lists |
| GET | `/galleries` · `/galleries/{slug}` | Galleries |
| GET | `/downloads?category=` | Downloads |
| GET | `/instagram` | Cached feed for the Instagram block |
| GET | `/redirects` | Full active redirect map for `proxy.ts` (cached, tag `redirects`) |
| POST | `/not-found` | Log a 404 (path, referrer). Called by Next.js |
| GET | `/sitemap/{type}` | URL + lastmod + images for `pages`, `products`, `collections`, `posts`, `jobs` |
| GET | `/preview/{type}/{id}?token=` | Draft content for preview mode |

## 2. Catalog

| Method | Path | Purpose |
|---|---|---|
| GET | `/collections` | Collection tree |
| GET | `/collections/{slug}` | Collection + facets + a page of products. Query: `page`, `per_page`, `sort` (one of `featured`, `newest`, `price_asc`, `price_desc`, `name`), filters such as `f[capacity_l]=500-1000`, `f[layers]=4,6`, `f[brand]=vectus` |
| GET | `/products/{slug}` | Full product: variants, options, grouped attributes, media, downloads, FAQs, related products, warranty, `purchase_mode`, assembly animation config |
| GET | `/products/{slug}/live` | Not cached: per-variant `price_paise`, `mrp_paise`, `in_stock` and effective `purchase_mode` |
| GET | `/compare?ids=12,15,31` | Products + aligned rows of comparable attributes (max 4 products) |
| GET | `/search?q=&type=products,posts,pages&page=` | Meilisearch-backed results |
| GET | `/search/suggest?q=` | Instant suggestions (top 6) |
| POST | `/products/{slug}/notify` | "Notify me when in stock" (`variant_id`, mobile or email) |

## 3. Blog & careers

| Method | Path | Purpose |
|---|---|---|
| GET | `/blogs/{blog}/posts?tag=&page=` | Post listing |
| GET | `/blogs/{blog}/posts/{slug}` | Post + author + related posts + related products |
| GET | `/authors/{slug}` | Author profile |
| GET | `/careers?department=&location=&type=` | Open jobs + filter options |
| GET | `/careers/{slug}` | Job detail (JobPosting JSON-LD is in `seo`) |
| POST | `/careers/{slug}/apply` · `/careers/apply` (general) | Multipart: fields + `resume` (PDF/DOC/DOCX, max 5 MB) + `turnstile_token` |

## 4. Leads & dealers

| Method | Path | Purpose |
|---|---|---|
| GET | `/locations/states` · `/locations/states/{id}/cities` | Dropdown data |
| POST | `/leads` | Enquiry (general, product or bulk). Body: `type`, contact fields, `product_id?`, `variant_id?`, `source_url`, `utm`, `consent`, `turnstile_token` |
| POST | `/dealer-applications` | Multipart dealer/distributor application |
| GET | `/dealers?state=&city=&pincode=&lat=&lng=&radius_km=&type=` | Dealer search (nearest first when lat/lng are given) |
| POST | `/newsletter` | Subscribe |

## 5. Customer auth & account

| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/otp/request` | `{ mobile, purpose }` → sends an OTP |
| POST | `/auth/otp/verify` | `{ mobile, otp, cart_token? }` → `{ token, customer }`; merges the guest cart |
| POST | `/auth/logout` | Revoke the token |
| GET, PATCH | `/account/profile` | Name, email, consents |
| GET, POST, PATCH, DELETE | `/account/addresses[/{id}]` | Address book |
| POST | `/account/delete-request` | DPDP data deletion request |

## 6. Cart & checkout

| Method | Path | Purpose |
|---|---|---|
| GET | `/cart` | Cart (by `X-Cart-Token` or customer) with totals, tax breakdown and shipping estimate |
| POST | `/cart/items` | `{ variant_id, qty }`. Returns 409 if the variant is in Enquiry mode or out of stock |
| PATCH, DELETE | `/cart/items/{id}` | Update quantity / remove |
| POST, DELETE | `/cart/coupon` | Apply / remove a coupon |
| POST | `/pincode/check` | `{ pincode, variant_ids[] }` → serviceable, ETA, COD, shipping charge |
| POST | `/checkout` | `{ address_id or address, billing, gstin? }` → creates a `pending_payment` order and a Razorpay order → `{ order_no, razorpay_order_id, amount, key }` |
| POST | `/checkout/verify` | `{ order_no, razorpay_payment_id, razorpay_order_id, razorpay_signature }` → marks the order paid |
| GET | `/account/orders` · `/account/orders/{order_no}` | Order history, detail, tracking |
| GET | `/account/orders/{order_no}/invoice` | Signed URL to the invoice PDF |
| POST | `/account/orders/{order_no}/cancel` | Allowed before shipping only |
| POST | `/account/orders/{order_no}/return` | Return request (if the policy allows) |

## 7. Warranty (R1 manual, R2 QR)

| Method | Path | Purpose |
|---|---|---|
| GET | `/qr/{code}` | Public. Resolves a scanned code to `state` (one of `invalid`, `inactive`, `unregistered`, `registered`, `blocked`) + `product` (name, image, variant, mfg date, warranty years) + `masked_owner` when registered. Never returns owner PII |
| POST | `/warranty/registrations` | Auth required. `code`, or (`product_id`, `variant_id`, `serial`) for manual mode; plus `purchase_date`, `dealer_id?`, `dealer_name?`, `invoice` (file), `installation_address`, `consent` |
| GET | `/account/warranties` · `/account/warranties/{registration_no}` | My products & warranty status |
| GET | `/account/warranties/{registration_no}/certificate` | Signed URL to the certificate PDF |
| POST | `/account/claims` | `{ registration_no, issue_type, description, photos[], preferred_visit_date? }` |
| GET | `/account/claims` · `/account/claims/{claim_no}` | Claim list / timeline |
| POST | `/account/claims/{claim_no}/messages` | Customer reply, with optional attachments |
| POST | `/account/claims/{claim_no}/rating` | Rate the resolution |

## 8. Inbound webhooks

| Path | From | Notes |
|---|---|---|
| `POST /webhooks/razorpay` | Razorpay | `payment.captured`, `payment.failed`, `refund.processed`. Verify `X-Razorpay-Signature`; idempotent by event ID |
| `POST /webhooks/shiprocket` | Shiprocket | Shipment status → order status + customer notification |
| `POST /webhooks/whatsapp` | WhatsApp BSP | Delivery status (optional) |

## 9. Outbound to Next.js

| Call | When |
|---|---|
| `POST https://www.vectus.in/api/revalidate` with `{ secret, tags[] }` | After any publishable model is saved, published, unpublished or deleted, and after redirect, menu or settings changes |
