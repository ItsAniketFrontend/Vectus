# E-commerce & Compare

## 1. Buy vs Enquiry (per product)

Each product has a `purchase_mode` of **Buy** or **Enquiry**. A variant can override it (e.g. small tank sizes buyable, large ones enquiry only).

A variant is **purchasable** only when all of these are true:

1. its effective mode (variant override, else product mode) is Buy
2. the store is enabled (a global kill switch that turns the whole site into enquiry mode)
3. it has a price
4. it's in stock (when stock is tracked)
5. the customer's pincode is serviceable (checked once a pincode is entered; until then the page shows "Check delivery")

### What the customer sees

| State | Product card | Product page buy box |
|---|---|---|
| Buy, purchasable | Price, struck-through MRP, **Add to Cart** | Variant picker, price incl. GST, qty, pincode check, **Add to Cart**, **Buy Now**, delivery ETA, "Bulk order? Enquire" link |
| Buy, out of stock | Price, "Out of stock" | **Notify me** (mobile/email) + **Enquire** |
| Buy, pincode not serviceable | — | "We don't deliver here yet" + **Enquire** + **Find a dealer** |
| Enquiry | "Price on request", **Enquire Now** | Variant picker (for specs), **Enquire Now** (form prefilled with product + variant), **Find a dealer**, **Download brochure** |
| All | **Add to Compare** | **Add to Compare**, warranty badge, specs, FAQs |

Buy Now = add to cart and go straight to checkout.

A likely split, for the client to confirm: small household plastics, bathware, taps, seat covers and garden pipes → Buy; water tanks, manholes, road-safety and bulk items → Enquiry (bulky freight, sold through dealers).

## 2. Cart

- Slide-out cart drawer + full `/cart` page.
- Guest cart via a token cookie, merged into the customer's cart on login.
- Line items, quantity steppers, remove, coupon, GST breakdown, shipping estimate by pincode, free-shipping progress bar, "You may also need" accessories.
- Re-validated on every change (purchasability, stock, price). The customer is told if a price changed.
- Abandoned carts of logged-in customers are listed in admin; optional WhatsApp/SMS reminder with consent.

## 3. Checkout

Single page, mobile-first:

1. **Mobile + OTP.** The account is created automatically — no passwords.
2. **Address.** Pincode auto-fills city/state; returning customers pick a saved address. Optional GSTIN for a business invoice.
3. **Delivery.** ETA and shipping charge from Shiprocket / rate table.
4. **Payment.** Razorpay Checkout: UPI, cards, netbanking, wallets, EMI. COD optional per pincode/order value.
5. **Confirmation.** `/order/{orderNo}` page + SMS/WhatsApp + email with the invoice.

The order is created as `pending_payment` before payment. The Razorpay webhook is the source of truth for "paid", which covers customers who close the tab. Unpaid orders expire after 30 minutes and release their stock.

## 4. Pricing & GST

- Prices are shown **inclusive of GST** (the B2C norm), stored per variant in paise, with MRP stored separately.
- Every product has an HSN code and a GST rate.
- Tax split: shipping to the same state as the dispatch location → CGST + SGST; otherwise IGST. The dispatch location is the configured warehouse state (or the nearest plant, if multi-warehouse comes later).
- A GST invoice PDF is generated on payment: sequential number per financial year (e.g. `VX/26-27/000123`), seller GSTIN, buyer GSTIN if given, HSN-wise tax summary.
- Coupons: percent, fixed or free shipping; minimum order, caps, per-customer limits, date range, collection/product restrictions.

## 5. Shipping

- **Shiprocket API:** pincode serviceability, ETA, rates by weight/dimensions, order push, AWB, label, pickup, tracking webhooks.
- Volumetric weight is calculated from variant dimensions — this matters for tanks and tubs.
- A fallback rate table (zone × weight slab) covers API downtime.
- Shipping policy page and a per-product "Ships in X days" line.
- Admin: create a shipment from an order, print the label, sync status, send the tracking link to the customer.

## 6. Orders (admin)

Status flow: `pending_payment → paid → processing → packed → shipped → delivered`, plus `cancelled`, `refunded` and `partially_refunded`.

Admin: filters, order detail with timeline, invoice re-generation, create shipment, cancel, full/partial refund (Razorpay API), notes, CSV export, daily sales widget. The customer is notified on every status change.

Returns: customer requests from their account (within the policy window) → admin approves or rejects → reverse pickup via Shiprocket → refund.

## 7. Customer account (`/account`)

One login (mobile OTP) for both the store and the warranty portal:

- Orders (status, tracking, invoice, cancel, return)
- Addresses
- My Products & Warranties (R2; R1 shows manual registrations)
- Service Claims (R2)
- Profile & consents, delete-my-data request

## 8. Compare

- **Add to Compare** on product cards and product pages. A sticky **compare tray** at the bottom of the screen shows thumbnails (max 4), "Compare now" and "Clear".
- Products can only be compared within the same attribute set (tank vs tank, tap vs tap). Adding a different type asks whether to start a new comparison.
- `/compare?ids=12,15,31` is a shareable URL; the selection is also kept in localStorage.
- Table: image, name, price or "Price on request", key badges, then grouped attribute rows (capacity, layers, UV layers, material, dimensions, warranty, ISI…). A **"Show differences only"** toggle. Product names stay pinned at the top while scrolling; on mobile the table swipes horizontally with the first column pinned.
- Each column has the right CTA (Add to Cart or Enquire) and "Remove".
- Only attributes marked `is_comparable` in admin appear.
- `/compare` is `noindex, follow` (thin, parameter-driven page). Comparisons worth ranking (e.g. "Ten-X vs Granito") can be published as normal CMS pages with a Compare block.
- GSAP Flip animates products flying into the tray and columns reordering.

## 9. Tracking

GA4 e-commerce events via the GTM data layer: `view_item_list`, `select_item`, `view_item`, `add_to_cart`, `remove_from_cart`, `view_cart`, `begin_checkout`, `add_shipping_info`, `add_payment_info`, `purchase`, `refund`. Custom events: `generate_lead` (enquiry), `add_to_compare`, `warranty_register`. Meta Pixel equivalents.

## 10. Needed from the client

- Razorpay merchant account (KYC). Refund, shipping, privacy, terms and contact pages must be live on the domain for approval
- Shiprocket account and pickup address(es)
- GSTIN, invoice details, HSN code + GST rate per product
- Price, MRP, weight and dimensions for every Buy-mode variant
- Which products are Buy vs Enquiry
- Return/refund and COD policy
