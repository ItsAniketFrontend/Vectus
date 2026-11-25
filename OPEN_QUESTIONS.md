# Open Questions for the Client

**Blocker** = work on that area can't start until it's answered. Record answers in the decision log at the bottom.

## A. QR codes & warranty (Release 2)

| # | Question | Why it matters | Priority |
|---|---|---|---|
| Q1 | Do products already carry QR codes? If yes, what does a scan open or contain today (URL, serial number, batch)? Please send photos of 3–4 labels | Existing stock must keep working with the new portal | **Blocker** |
| Q2 | Should each unit get a unique QR (recommended), or one QR per product model? | Decides the whole warranty design (WARRANTY_PORTAL.md §1) | **Blocker** |
| Q3 | How are labels printed today — at the plant (which printer?) or by a label vendor? Can they print variable data (unique codes)? | Determines the label export format | **Blocker** |
| Q4 | Is there a serial/batch number system in the ERP? Which ERP (SAP, Tally, other)? | Generate codes ourselves vs import from ERP | High |
| Q5 | Warranty terms per product line: years, full vs pro-rata, invoice mandatory or not, registration deadline? | Warranty policies & certificate | High |
| Q6 | Who handles claims — a central service team, regional teams, or dealers? Expected SLAs? | Claim assignment & SLA rules | High |
| Q7 | What must the warranty certificate show (logo, signatory, terms, QR)? | Certificate PDF design | Medium |

## B. E-commerce

| # | Question | Why it matters | Priority |
|---|---|---|---|
| Q8 | Which products/variants are Buy vs Enquiry at launch? | Product setup, pricing data | **Blocker** for commerce |
| Q9 | Price, MRP, weight, dimensions, HSN code and GST rate for every Buy variant | Checkout, tax, shipping | High |
| Q10 | Ship from one warehouse or several plants? Pickup addresses? | Shipping & GST origin | High |
| Q11 | Payment gateway: is Razorpay OK, or is there an existing PayU/CCAvenue account? Is COD allowed? | Integration | High |
| Q12 | Courier: is Shiprocket OK, or are there existing courier contracts? | Integration | High |
| Q13 | Is stock managed in the ERP (sync needed) or manually in admin? | Stock module scope | High |
| Q14 | Return / refund / cancellation policy | Policies, order flow | High |
| Q15 | Is the newsletter offer ("10% off + free shipping on first purchase") staying? | Coupons | Low |

## C. Content, SEO & design

| # | Question | Why it matters | Priority |
|---|---|---|---|
| Q16 | Access to Shopify admin (for export), Google Search Console, GA4 and GTM | Migration & baseline | **Blocker** for Phase 0 |
| Q17 | Brand guidelines, logo files, fonts, colours | Design system | High |
| Q18 | CAD/3D files for the products to animate — and which 4–6 products? | Assembly animations | High |
| Q19 | Product photography: reuse the Shopify images or do a new shoot? | Visual quality | Medium |
| Q20 | Hindi or other language versions — now, later or never? | Routing & hreflang | Medium |
| Q21 | Should the 7 brands (Vectus, Gangotri, Citizen, Premier, Ganga, Waterwell, Maali) get their own brand pages? | Sitemap & SEO | Medium |
| Q22 | Who on the SEO team will use the admin, and what tools/workflows do they already use? | Admin UX, training | Medium |
| Q23 | The duplicate pages/collections marked "Review" in PAGE_INVENTORY.md — SEO team to confirm the targets | Redirect map | High |

## D. Teams, integrations & hosting

| # | Question | Why it matters | Priority |
|---|---|---|---|
| Q24 | Lead routing: which states go to which team/email? Is there a CRM (Salesforce, Zoho, LeadSquared) to push leads into? | Leads module | Medium |
| Q25 | Source of the dealer list (Excel/ERP) with addresses, and how often it changes | Dealer locator | Medium |
| Q26 | HR: who receives applications for each department? Is interview scheduling needed? | Careers module | Medium |
| Q27 | SMS provider & DLT registration: existing sender ID/templates? WhatsApp Business API account? | OTP & notifications | High |
| Q28 | Hosting: the client's own AWS/Azure account, or do we provide it? Who maintains the site after launch (AMC)? | Infrastructure & handover | High |
| Q29 | Is any link to the employee portal (vectusconnect.vectus.in) needed beyond the footer link? | Scope | Low |
| Q30 | Target launch date or business deadlines (season, trade show)? | Timeline | High |

## Decision log

| Date | Decision | Decided by | Notes |
|---|---|---|---|
| 2026-11-25 | Frontend Next.js; backend Laravel (PHP) + MySQL; admin in Filament | Proposed | Client to confirm Laravel |
| 2026-11-25 | Keep Shopify URL patterns at launch | Proposed | Reduces SEO risk |
| 2026-11-25 | Two releases: R1 website & store, R2 warranty portal | Proposed | |
| 2026-11-25 | Buy/Enquiry set per product, with per-variant override | Client requirement | |
| 2026-11-25 | Add to Compare on products | Client requirement | |
| 2026-11-25 | GSAP + Lenis animation, product assembly animations, modern UI/UX | Client requirement | |
