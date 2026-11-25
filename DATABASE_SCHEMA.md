# Database Schema (MySQL 8)

First-pass schema for planning. Column lists show the important fields; the Laravel migrations will be the final source of truth.

## Conventions

- InnoDB, `utf8mb4_unicode_ci`.
- Every table has `id BIGINT UNSIGNED PK AUTO_INCREMENT`, `created_at`, `updated_at`. Content tables also have `deleted_at` (soft deletes).
- Publishable content: `status ENUM('draft','published','archived')` and `published_at DATETIME NULL` (a future date means scheduled).
- Money: integer paise (`*_paise`). Percentages: basis points (`*_bp`, 1800 = 18%). Dates: UTC.
- Polymorphic relations use `{name}_type` + `{name}_id` (Laravel morphs).
- JSON columns hold flexible blocks/config. Anything we filter or join on gets a real column.

## Entity overview

```
collections ─< collection_product >─ products ─< product_variants ─< product_units (QR, R2)
     │                                  │  │                              │
     └ parent_id (tree)                 │  └< product_attribute_values    └─ warranty_registrations ─< warranty_claims
                                        │        └ attributes (attribute_sets)
                                        └ brand_id → brands
customers ─< orders ─< order_items → product_variants
customers ─< carts ─< cart_items
seo_meta  (morph → pages, products, collections, posts, job_openings, …)
redirects · schema_templates · meta_templates · settings · menus ─< menu_items
posts → blogs, authors, tags        job_openings → departments ─< job_applications
leads · dealers · dealer_applications · states ─< cities
```

## 1. Staff, roles, system

```
users                      staff accounts (Filament)
  name, email UNIQUE, password, two_factor_secret, two_factor_recovery_codes,
  is_active, last_login_at
roles, permissions, model_has_roles, model_has_permissions, role_has_permissions
                           spatie/laravel-permission
activity_log               spatie/laravel-activitylog (subject, causer, properties JSON)
content_revisions          snapshot on every save
  revisionable_type, revisionable_id, user_id, data JSON, note
settings
  group (general|contact|social|seo|commerce|integrations|notifications), key, value JSON
  UNIQUE(group, key)
media                      spatie/laravel-medialibrary
  model_type, model_id, collection_name, disk, file_name, mime_type, size,
  custom_properties JSON (alt, caption, focal point), responsive_images JSON
notification_logs
  channel (sms|whatsapp|email), recipient, template, payload JSON, status,
  provider_message_id, error, related_type, related_id
```

## 2. CMS

```
pages
  title, slug UNIQUE, template (default|landing|legal), blocks JSON,
  parent_id NULL, status, published_at, show_in_sitemap BOOL
menus
  location UNIQUE (header|footer_about|footer_products|footer_quick|mobile)
menu_items
  menu_id, parent_id NULL, label, url NULL, linkable_type NULL, linkable_id NULL,
  anchor NULL, open_in_new_tab, badge NULL, sort_order
brands
  name, slug UNIQUE, logo (media), description, website_url, sort_order, is_active
client_logos
  name, logo (media), url NULL, sort_order
galleries
  title, slug UNIQUE, description, status            (items = media collection "items")
downloads
  title, category (brochure|catalogue|certificate|manual|price_list|other),
  file (media), product_id NULL, collection_id NULL, sort_order, is_active
testimonials (optional)
  name, designation, city, quote, photo (media), rating, sort_order
```

Shape of `pages.blocks`:

```json
[
  {
    "id": "b_9f2c",
    "type": "hero_video",
    "anchor": null,
    "visibility": { "desktop": true, "mobile": true },
    "animation": { "preset": "split-text-reveal", "delay": 0 },
    "data": {
      "heading": "Built to perform. Trusted to last.",
      "subheading": "…",
      "video_media_id": 812,
      "poster_media_id": 813,
      "cta": { "label": "Show more", "url": "/collections/water-tanks" }
    }
  }
]
```

## 3. SEO

```
seo_meta                       one row per SEO-able entity (morph)
  seoable_type, seoable_id     UNIQUE together
  meta_title NULL, meta_description NULL, h1_override NULL,
  canonical_url NULL, robots_index BOOL DEFAULT 1, robots_follow BOOL DEFAULT 1,
  og_title NULL, og_description NULL, og_image_media_id NULL,
  twitter_card ENUM('summary','summary_large_image'),
  focus_keyword NULL, secondary_keywords JSON NULL,
  breadcrumb_label NULL,
  schema_mode ENUM('auto','auto_plus_custom','custom_only') DEFAULT 'auto',
  custom_schema JSON NULL      one or more JSON-LD objects
  sitemap_include BOOL DEFAULT 1, sitemap_priority DECIMAL(2,1) NULL,
  last_audit JSON NULL         cached audit warnings
meta_templates                 fallback title/description patterns per entity type
  entity_type UNIQUE, title_pattern, description_pattern
schema_templates
  entity_type (product|collection|page|post|job|dealer|global), name,
  template JSON (with {{tokens}}), is_active, priority
redirects
  from_path UNIQUE (normalised path, no domain), to_path, status_code (301|302|410),
  match_type ENUM('exact','prefix','regex') DEFAULT 'exact',
  source ENUM('manual','slug_change','import','404_fix'),
  hits INT, last_hit_at, is_active, notes
not_found_logs                 404s reported by Next.js
  path UNIQUE, referrer, hits, last_seen_at, resolved BOOL
tracking_scripts
  name, placement (head|body_start|body_end), code TEXT, is_active, consent_category
```

## 4. Catalog

```
collections
  name, slug UNIQUE, type ENUM('category','series','curated','brand'),
  parent_id NULL, brand_id NULL, short_description, description (rich),
  banner (media), attribute_set_id NULL, blocks JSON NULL (extra content below the grid),
  default_sort, status, sort_order, shopify_id NULL
collection_product
  collection_id, product_id, is_primary BOOL, sort_order    PK(collection_id, product_id)
products
  brand_id, name, slug UNIQUE, tagline, short_description, description (rich),
  primary_collection_id, attribute_set_id,
  purchase_mode ENUM('buy','enquiry') DEFAULT 'enquiry',
  warranty_policy_id NULL, warranty_years NULL,
  hsn_code, gst_rate_bp,
  badge NULL ("Best seller", "New"),
  is_featured, is_comparable DEFAULT 1, status, published_at, sort_order,
  assembly_animation JSON NULL      { type: sequence|model|layers, steps, settings }
  shopify_id NULL, shopify_handle NULL
product_options                  e.g. Capacity, Colour, Layers
  product_id, name, sort_order
product_option_values
  product_option_id, value, swatch_hex NULL, sort_order
product_variants
  product_id, sku UNIQUE, title ("1000 L / Blue"), barcode NULL,
  mrp_paise NULL, price_paise NULL,
  purchase_mode_override ENUM('buy','enquiry') NULL,
  track_stock BOOL, stock_qty INT NULL, low_stock_threshold,
  weight_grams, length_mm, width_mm, height_mm,
  is_default, is_active, sort_order, shopify_variant_id NULL
variant_option_values
  variant_id, option_value_id      PK(variant_id, option_value_id)
attribute_sets                   "Water tank", "Tap", "Toilet seat cover", …
  name, slug
attributes
  code UNIQUE (capacity_l, layers, uv_layers, material, colour, height_mm, isi_mark, …),
  label, type ENUM('text','number','boolean','select','multiselect'),
  unit NULL, group_name (Dimensions, Material, Warranty, …),
  is_filterable, is_comparable, is_visible_on_pdp, options JSON NULL, sort_order
attribute_set_attribute
  attribute_set_id, attribute_id, sort_order
product_attribute_values
  product_id, variant_id NULL, attribute_id,
  value_text NULL, value_number DECIMAL(12,3) NULL, value_bool NULL
  INDEX(attribute_id, value_number), INDEX(attribute_id, value_text)
product_relations
  product_id, related_product_id, type ENUM('related','upsell','accessory','compare_suggestion'), sort_order
product_faqs
  product_id, question, answer, sort_order
stock_notifications              "Notify me" requests for out-of-stock variants
  variant_id, mobile NULL, email NULL, notified_at NULL
reviews (optional, phase 3)
  product_id, customer_id NULL, rating TINYINT, title, body, status (pending|approved|rejected)
```

Product media collections: `images` (ordered, with alt text), `gallery_360`, `assembly_frames`, `model_3d`, `brochure`, `video`.

## 5. Commerce

```
customers
  name NULL, mobile VARCHAR(15) UNIQUE, email NULL, mobile_verified_at,
  marketing_consent BOOL, consent_at, last_login_at, is_blocked
customer_addresses
  customer_id, label, name, phone, line1, line2, landmark, city, state_id, pincode,
  gstin NULL, is_default_shipping, is_default_billing
otp_requests
  mobile, code_hash, purpose (login|warranty), attempts, expires_at, verified_at, ip
carts
  token UNIQUE (guest), customer_id NULL, coupon_id NULL, pincode NULL, expires_at
cart_items
  cart_id, variant_id, qty         UNIQUE(cart_id, variant_id)
orders
  order_no UNIQUE (e.g. VX-2026-000123), customer_id,
  status ENUM('pending_payment','paid','processing','packed','shipped','delivered',
              'cancelled','refunded','partially_refunded'),
  payment_status ENUM('pending','paid','failed','refunded','partial_refund'),
  subtotal_paise, discount_paise, shipping_paise, tax_paise, total_paise,
  coupon_code NULL, shipping_address JSON, billing_address JSON, gstin NULL,
  place_of_supply_state_id, notes NULL, utm JSON NULL, placed_at, expires_at NULL
order_items
  order_id, variant_id, product_name, variant_title, sku, hsn_code, qty,
  unit_price_paise, discount_paise, tax_rate_bp, cgst_paise, sgst_paise, igst_paise, line_total_paise
order_status_history
  order_id, from_status, to_status, user_id NULL, note
payments
  order_id, gateway (razorpay), gateway_order_id, gateway_payment_id, method,
  amount_paise, status, raw JSON
refunds
  order_id, payment_id, amount_paise, reason, gateway_refund_id, status, user_id
returns
  order_id, items JSON, reason, photos (media), status (requested|approved|rejected|picked_up|refunded)
shipments
  order_id, provider (shiprocket), provider_order_id, awb, courier, status,
  tracking_url, shipped_at, delivered_at, raw JSON
invoices
  order_id UNIQUE, invoice_no UNIQUE (financial-year series), issued_at, pdf (media)
coupons
  code UNIQUE, type ENUM('percent','fixed','free_shipping'), value, min_order_paise,
  max_discount_paise, usage_limit, per_customer_limit, applies_to JSON (collections/products),
  starts_at, ends_at, is_active
coupon_redemptions
  coupon_id, order_id, customer_id
serviceable_pincodes             cache of Shiprocket results + manual overrides
  pincode UNIQUE, is_serviceable, cod_available, eta_days, checked_at
newsletter_subscribers
  email UNIQUE, status, source, consent_at
```

## 6. Leads & dealers

```
states       name, code (GST state code)
cities       state_id, name
leads
  type ENUM('general','product','bulk','contact','warranty_query'),
  first_name, last_name, email, phone, state_id, city_id,
  query_type (General Inquiry|Order Support|Product Question|Returns & Exchanges|Complaint|Feedback|Other),
  message, product_id NULL, variant_id NULL, quantity NULL, pincode NULL,
  source_url, utm JSON, ip, user_agent,
  status ENUM('new','contacted','qualified','converted','closed','spam'),
  assigned_to (users.id) NULL, consent_at
  INDEX(status, assigned_to)
lead_notes   lead_id, user_id, note
lead_routing_rules
  state_id NULL, lead_type NULL, assign_to_user_id, notify_emails JSON, priority
dealers
  name, type ENUM('distributor','dealer','retailer','service_centre'),
  contact_person, phone, email, address, city_id, state_id, pincode,
  latitude DECIMAL(10,7), longitude DECIMAL(10,7), brands JSON, product_lines JSON,
  is_active
  INDEX(state_id, city_id), INDEX(pincode)
dealer_applications
  business_name, contact_person, phone, email, gstin, pan, address, city_id, state_id,
  pincode, business_type, years_in_business, current_brands, warehouse_sqft,
  territory_requested, documents (private media), status (new|reviewing|approved|rejected),
  assigned_to NULL, notes
```

## 7. Blog

```
blogs         name, slug UNIQUE (news|publications)
authors       name, slug UNIQUE, designation, bio, photo (media), linkedin_url, user_id NULL
posts
  blog_id, author_id, title, slug, excerpt, content (rich / blocks JSON),
  featured_image (media), reading_minutes, status, published_at, is_featured,
  shopify_id NULL                       UNIQUE(blog_id, slug)
tags          name, slug UNIQUE
post_tag      post_id, tag_id
post_product  post_id, product_id        related products shown in the post
post_faqs     post_id, question, answer, sort_order
```

## 8. Careers

```
departments    name, slug
job_openings
  title, slug UNIQUE, department_id, location_city, location_state, is_remote,
  employment_type ENUM('full_time','part_time','contract','internship'),
  experience_min, experience_max, salary_min_paise NULL, salary_max_paise NULL, show_salary,
  openings_count, summary, description (rich), responsibilities (rich), requirements (rich),
  benefits (rich) NULL, valid_through DATE, status, published_at,
  notify_emails JSON, hiring_manager_id NULL
job_applications
  job_opening_id NULL (NULL = general application), name, email, phone,
  current_city, total_experience, current_company, current_ctc NULL, expected_ctc NULL,
  notice_period_days NULL, linkedin_url NULL, cover_letter NULL, resume (private media),
  status ENUM('new','screening','shortlisted','interview','offered','hired','rejected','on_hold'),
  rating TINYINT NULL, source, consent_at, retain_until DATE
application_status_history   application_id, from_status, to_status, user_id, note
application_notes            application_id, user_id, note
```

## 9. Warranty & QR (Release 2)

```
plants        name, code UNIQUE (e.g. GWL, JPR), city, state_id
warranty_policies
  name, years, type ENUM('full','pro_rata'), pro_rata_rules JSON NULL,
  terms (rich), requires_invoice BOOL, registration_window_days NULL
qr_batches
  batch_no UNIQUE, variant_id, plant_id, mfg_date, quantity,
  status ENUM('generated','exported','printed','activated','void'),
  export_file (media), created_by, activated_at
product_units                     one row per physical product (per-unit QR model)
  code CHAR(11) UNIQUE            10 random Crockford Base32 chars + 1 check char
  batch_id, variant_id, serial_no NULL, mfg_date,
  status ENUM('inactive','active','registered','blocked','replaced'),
  first_scanned_at NULL, scan_count INT DEFAULT 0
  INDEX(batch_id), INDEX(status)
legacy_codes                      codes printed before R2 (format TBD)
  raw_value UNIQUE, product_id NULL, variant_id NULL, product_unit_id NULL
qr_scans                          high volume → monthly partitions or 12-month retention
  product_unit_id NULL, code, scanned_at, ip, user_agent, city NULL, customer_id NULL, outcome
warranty_registrations
  registration_no UNIQUE, product_unit_id UNIQUE NULL, variant_id, customer_id,
  serial_entered NULL (manual mode), purchase_date, dealer_id NULL, dealer_name NULL,
  invoice (private media), installation_address JSON, pincode,
  warranty_policy_id, warranty_start, warranty_end,
  status ENUM('pending_verification','active','rejected','expired','void'),
  verified_by NULL, verified_at NULL, rejection_reason NULL,
  certificate (media), channel ENUM('qr','manual','admin')
warranty_claims
  claim_no UNIQUE, warranty_registration_id, customer_id,
  issue_type (leakage|crack|discolouration|fitting|other), description, photos (private media),
  preferred_visit_date NULL,
  status ENUM('open','assigned','visit_scheduled','in_progress','resolved','rejected','closed'),
  priority, assigned_to NULL, dealer_id NULL (service partner),
  sla_due_at, resolution NULL, resolution_type (repair|replace|refund|rejected) NULL,
  customer_rating TINYINT NULL, closed_at
claim_updates
  claim_id, user_id NULL, customer_id NULL, status NULL, message, attachments (media), is_public
```

## 10. Indexes & volume

- `product_units` can reach millions of rows a year if every unit gets a QR. `code` is a short CHAR with a unique index, so lookups stay fast at that size. Codes are generated in batches by queued jobs.
- `qr_scans` grows fastest: partition it by month, or prune after 12 months and keep aggregates.
- Full-text product search goes through Meilisearch, not MySQL `LIKE`.
- Composite indexes for collection filtering (`product_attribute_values(attribute_id, value_number)`), order lists (`orders(customer_id, placed_at)`) and lead routing (`leads(status, assigned_to)`).
