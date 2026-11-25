# Warranty & Customer Portal (Release 2)

Every Vectus product carries a QR code. A customer scans it, sees that the product is genuine, registers the warranty in under a minute and gets a digital certificate. Later they can raise a service claim from the same account.

## 1. First decision: what does the QR code identify?

**Blocker — must be answered before design** (OPEN_QUESTIONS.md Q1–Q4). There are two possible models:

| | **A. Unique code per unit** (recommended) | **B. One code per product model** |
|---|---|---|
| What's printed | A different QR on every tank/product | The same QR on all units of a SKU |
| Scan shows | That exact unit: model, size, plant, mfg date, warranty status | Product page + registration form |
| Serial number | Inside the code — nothing to type | Customer types the serial/batch printed on the product |
| Genuine-product check | Yes | No |
| Duplicate / fraudulent registration | Blocked: one registration per code | Hard to prevent |
| Production impact | Needs variable QR labels per unit (label printer on the line, or pre-printed label rolls from our batches) | Static artwork, no change |
| Data volume | Millions of codes per year | One per SKU |

**If products already carry QR codes**, we need to know what they encode (URL? serial? batch?) and support them through a legacy resolver so existing stock keeps working.

The rest of this document assumes **Model A**, with Model B supported as the "manual" path (also used in R1).

## 2. Code format

- 10 random characters from Crockford Base32 (`0–9`, `A–Z` without I, L, O, U) + 1 check character, e.g. `VX7K2M9QPA3`. About 10¹⁵ combinations, so codes can't be guessed and nobody can enumerate other people's products.
- QR content: `https://vectus.in/w/VX7K2M9QPA3`. A short URL keeps the QR small and readable on curved plastic.
- A human-readable code is printed under the QR for manual entry; the check character catches typos.
- Codes are random, not sequential IDs. Batch details live in the database, not in the code.

## 3. Label generation (admin)

1. A service/production manager creates a **QR batch**: product variant, plant, manufacturing date, quantity.
2. A queue job generates the codes (`product_units`, status `inactive`).
3. Export for the printer: CSV (code + URL + human-readable fields) and/or print-ready PDF label sheets. Size and layout are agreed with production (e.g. 40×40 mm).
4. The batch is marked **printed**, then **activated** when it leaves the plant. Only active codes can be registered, which protects against stolen labels.
5. Misprinted batches can be voided.

Possible later additions: a production-line scanner app that marks each unit as produced; ERP sync for dispatch to dealers.

## 4. Customer flow

```
Scan QR (phone camera)
   │
   ▼
/w/{code}  ── invalid code ──► "We couldn't verify this code" + manual entry + support contact
   │
   ├─ inactive ─────────────────► "Product not yet activated" + support
   ├─ blocked ──────────────────► "Contact customer care"
   ├─ registered to someone else ► "Already registered" (masked: "+91 98•••••12") + "Not you? Contact us"
   ├─ registered to me ─────────► Warranty card: status, expiry, certificate, raise claim
   │
   └─ unregistered (genuine) ───► Product verified ✔ (image, model, capacity, plant, mfg date, warranty years)
                                       │
                                       ▼  "Register warranty"
                                Mobile + OTP (skipped if already logged in)
                                       │
                                       ▼
                                Purchase date · Dealer (search the list or type a name) ·
                                Invoice photo/PDF (if the policy requires it) ·
                                Installation address & pincode · Consent
                                       │
                                       ▼
                                Registration created → certificate PDF
                                SMS + WhatsApp + email with the certificate link
```

The genuine-product screen gets a short "verified" animation and nothing heavier, because customers open it on mobile data straight from the camera.

Manual path: `/pages/warranty-registration` → enter the printed code (Model A), or pick the product and enter its serial (Model B / legacy).

## 5. Warranty rules

- Warranty policies per product line (e.g. tanks 10–15 years depending on the model; taps and bathware differ): full vs pro-rata, terms text, whether an invoice is required, registration window (e.g. within 90 days of purchase).
- Warranty start = purchase date (checked against the invoice), or mfg date + a grace period when there's no invoice. The client decides.
- Registrations with an invoice can activate automatically. Suspicious ones go to a **verification queue**: purchase date before mfg date, unreadable invoice, location far from the dealer, too many registrations from one number.

## 6. Customer portal (`/account`)

| Section | Features |
|---|---|
| My Products | All registered units: image, model, code, purchase date, warranty end, status badge; "Add product" (scan or enter code) |
| Warranty detail | Certificate download, terms, dealer, invoice, installation address |
| Raise a claim | Pick product → issue type → description → photos/video → preferred visit date → submit; claim number sent by SMS/WhatsApp |
| My Claims | Timeline of updates (assigned, visit scheduled, resolved), reply with messages/photos, rate the resolution |
| Orders | From the online store (ECOMMERCE.md) |
| Profile | Name, email, addresses, consents, delete-my-data |

## 7. Claims handling (admin)

- **Claims board** (Kanban by status) and table, filterable by region, product, age and SLA.
- **Auto-assignment** by pincode/state to a regional service manager or service partner (a dealer of type `service_centre`).
- **SLA timers** (e.g. first response in 24 h, visit within 72 h) with overdue highlighting and escalation emails.
- Internal notes vs public updates (public updates notify the customer).
- Resolution: repair, replace (links a replacement unit; the old code becomes `replaced`), or reject with a reason.
- Reports: registrations per product/plant/month, claim rate per batch (early warning on manufacturing defects), resolution times, customer ratings.

## 8. Anti-fraud & abuse

- One registration per code; ownership transfers only through customer care.
- Codes must be `active` (activated batches only).
- Rate limits on `/qr/{code}` and OTP; Turnstile on manual entry.
- Scan log (time, approximate location, device), with alerts when one code is scanned from many cities or one IP tries many codes.
- Public endpoints never return PII; the owner is shown masked.
- Invoices and photos sit in private storage, served only through signed URLs.

## 9. Notifications

| Event | SMS | WhatsApp | Email |
|---|---|---|---|
| OTP | ✔ | fallback | — |
| Warranty registered (+ certificate) | ✔ | ✔ | ✔ |
| Registration needs info / rejected | ✔ | ✔ | ✔ |
| Claim created / assigned / visit scheduled / resolved | ✔ | ✔ | ✔ |
| Warranty expiring (90 days before) | — | ✔ (with consent) | ✔ |

SMS templates must be registered on the DLT portal (TRAI) before go-live; WhatsApp templates are approved through the BSP.

## 10. Scale

- A unique index on `product_units.code` keeps lookups fast at tens of millions of rows.
- Codes are generated in chunks (e.g. 10,000 per job).
- `qr_scans` is partitioned by month or pruned after 12 months.
- The `/w/{code}` page is lightweight and rendered dynamically; its product data is cached.

## 11. Delivery plan (≈5–6 weeks)

1. Week 1: finalise the QR model, label spec with production, warranty policies, legacy code format
2. Week 2: batches, code generation, label export, `/qr/{code}` API
3. Week 3: scan page, registration flow, OTP, certificate PDF
4. Week 4: customer portal products/claims, claims admin, assignment, SLA
5. Week 5: notifications, reports, anti-fraud rules, load test
6. Week 6: pilot with one plant and one product batch → fixes → rollout
