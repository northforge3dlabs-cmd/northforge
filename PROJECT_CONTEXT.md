# Northforge Labs — Project Context

## What is this?

Northforge Labs is a Canadian 3D printing hobby business that sells
storage and utility products for tabletop miniature painters (e.g.
paint racks, carry cases, trays). Products are made-to-order in PLA
filament with customer-selectable accent colours. The store is live
at **northforge-labs.myshopify.com**.

---

## Tech Stack

| Layer | Detail |
|---|---|
| Platform | Shopify (Dawn theme, heavily customized) |
| Theme repo | GitHub: `northforge3dlabs-cmd/northforge` (branch: `main`) |
| Admin scripts | `C:\Users\clock\OneDrive\Desktop\northforge-upload\` (Node.js scripts using Shopify Admin REST API) |
| Local theme path | `C:\Users\clock\OneDrive\Desktop\northforge\` |
| Custom CSS | `assets/northforge-custom.css` |

---

## Deployment Workflow

### Theme files (CSS, Liquid)
**Do NOT rely on GitHub push alone** — Shopify's GitHub integration
does not reliably auto-deploy. Always use Shopify CLI:

```bash
# Push a single file
shopify theme push --theme 144118120522 --only assets/northforge-custom.css --allow-live

# Push multiple files
shopify theme push --theme 144118120522 --only assets/northforge-custom.css --only layout/theme.liquid --allow-live

# Push everything
shopify theme push --theme 144118120522 --allow-live
```

Live theme name: `northforge/main` | Theme ID: `144118120522`

### Page content (HTML body of Shopify pages)
Page content is managed via Node.js scripts in the
`northforge-upload` folder. These use the Shopify Admin OAuth API.
Run from terminal:

```bash
cd "C:\Users\clock\OneDrive\Desktop\northforge-upload"
node fix-faq-accordion.js    # updates FAQ page
node fix-about-page.js       # updates About page
# etc.
```

Each script opens a browser OAuth window. After approving, it pushes
content via the Shopify Admin API and prints a success confirmation.

---

## Important CSS Notes

### Root font size
The theme sets `html { font-size: calc(var(--font-body-scale) * 62.5%) }`
which makes **`1rem = 10px`** (at the default body scale of 100).
Always calculate sizes with this in mind:

| rem value | actual px |
|---|---|
| 1rem | 10px |
| 1.5rem | 15px (mobile body text) |
| 1.6rem | 16px (desktop body text) |
| 2rem | 20px |
| 2.2rem | 22px |

### northforge-custom.css loading caveat
`northforge-custom.css` is loaded inside
`{%- if settings.cart_type == 'drawer' -%}` in `layout/theme.liquid`.
The store uses `cart_type: notification`, so **this file does NOT
load globally**. Do not rely on it for page-specific styles. Instead,
embed `<style>` tags directly in page HTML content via the Admin
API scripts.

### Theme font
Body and heading font: **Assistant** (Google Fonts), loaded at
weight 400 (`assistant_n4`). Bold weight (700) is synthesized
by the browser.

---

## Brand Identity

| Token | Value |
|---|---|
| Primary | Deep Forest Green `#2D4A3E` |
| Accent | Aged Brass `#C4922A` |
| Background | Warm Off-White `#F5F0E8` |
| Text | Charcoal Brown `#2E2318` |
| Soft | Warm Sage `#8FAF8A` |
| Card | Light Linen `#EDE8DF` |
| Border | `#D4CEC4` |
| Heading font | Playfair Display (Bold) |
| Body font | Lato (Regular/Light) |

Note: Theme currently uses Assistant font (loaded via Dawn).
Playfair Display + Lato are the brand specification fonts —
apply via northforge-custom.css or inline styles as needed.

---

## Pages

| Page | Handle | Shopify ID |
|---|---|---|
| Frequently Asked Questions | `frequently-asked-questions` | 115889438794 |
| About Northforge Labs | `about-northforge-labs` | 115888455754 |
| Contact | `contact` | 115104907338 |

---

## Products

| Product | Trays | Base Price | Max Price | Colours |
|---|---|---|---|---|
| War-Ganizer 3.0 Skirmish Bundle | 3 | $95 CAD | $110 CAD | Red, Blue, Yellow, Purple, Black |
| War-Ganizer 3.0 Battle Bundle | 4 | $125 CAD | $145 CAD | Red, Blue, Yellow, Purple, Black |
| War-Ganizer 3.0 Standard Bundle | TBD | TBD | TBD | TBD |
| War-Ganizer 3.0 Commander Bundle | TBD | TBD | TBD | TBD |

- Body colour: Matte black PLA (all variants)
- Licensed from 3 Five Design (war-ganizer.ca)
- Made to order — inventory tracking disabled
- Continue selling when out of stock: enabled
- Estimated dimensions: 12"x12"x5" packed (~1.5kg actual,
  ~2.34kg dimensional weight)

### Tray Configuration System

Each bundle offers per-tray type selection. Paint trays add $5 CAD each.

**Tray options (per tray):**
- Magnet Tray (no upcharge)
- GW Tray (no upcharge)
- Paint Tray — Vallejo / Army Painter (+$5 CAD)
- Paint Tray — Citadel Paints (+$5 CAD)

**Shopify variant structure (per product):**
- Option 1: `Color` — the accent colour
- Option 2: `Paint Trays` — count of paint trays selected (0 to N),
  drives pricing. Values: `0`, `1`, `2`, ... up to tray count.
- Per-tray type and bottle format are captured as line item
  properties (`Tray 1`, `Tray 2`, etc.) — visible in Shopify orders
  for fulfillment.

**Tray count tag:** Each product must have a tag `tray-count-N`
(e.g. `tray-count-3`) so the theme renders the correct number of
tray dropdowns. Default fallback is 3 if tag is absent.

**Theme file:** `snippets/product-variant-picker.liquid`
— tray config UI, hidden Paint Trays select, and sync JS are all
  in this file, scoped to products that have a `Paint Trays` option.

**Admin scripts for tray/variant setup:**
| Script | Purpose |
|---|---|
| `fix-skirmish-variants.js` | Initial Skirmish Bundle variant restructure |
| `fix-skirmish-colors.js` | Added Purple + Black to Skirmish Bundle |
| `fix-battle-variants.js` | Sets up Battle Bundle (and re-tags Skirmish) |
| `fix-battle-price.js` | Sets Battle Bundle base price to $125 |

---

## Payment Infrastructure

| Method | Status | Notes |
|---|---|---|
| Shopify Payments | ✅ Active | Primary processor |
| Shop Pay | ✅ Active | Auto-enabled with Shopify Payments |
| Shop Pay Installments | ✅ Active | US customers only |
| Apple Pay | ✅ Active | |
| Google Pay | ✅ Active | |
| Visa / Mastercard / Amex | ✅ Active | |
| Diners Club / Discover | ✅ Active | |
| PayPal | ✅ Active | Verified. Weekly auto-payout to Wise pending setup |

### Payout Flow
- Shopify Payments → Wise CAD account (weekly, confirmed)
- PayPal → Wise CAD account (weekly Friday auto-transfer —
  pending final verification of micro-deposits in Wise)

### Important
- Shopify Payments and standalone Stripe account are separate
- Do NOT use external Stripe through Shopify — adds transaction
  fees on top of processing fees
- Shopify Payments is the correct payment processor for the store

---

## Banking

| Account | Purpose |
|---|---|
| Wise CAD business account | Primary business banking |
| Wise multi-currency | Available for USD/international if needed |

All payment processor payouts route to Wise CAD account.

---

## Shipping

- App: netParcel (Shopify-integrated, free to install)
- Carriers: multi-carrier rate comparison (Canada Post, UPS,
  Purolator, FedEx) — best rate selected per shipment
- Zones: Canada, USA, International
- Rates: customer-paid, calculated at checkout (real-time)
- No Canada Post business account — accessing CP rates via
  netParcel's negotiated volume rates
- Label printer: Rollo or Dymo 4XL (pending purchase)
- Postal scale: pending purchase

---

## Work Completed (as of March 20, 2026, updated)

### Infrastructure
- Shopify store live at northforge-labs.myshopify.com
- Stripe / Shopify Payments active
- Wise banking active
- PayPal active (weekly auto-payout setup in progress)
- netParcel installed and configured
- Accounting workbook complete

### Brand Assets (complete)
- Logo horizontal (green + white)
- Logo stacked (green + white)
- Wordmark
- Favicon (512x512px)
- Color palette and typography defined

### Store Content (all written and implemented)
- About page (first-person, geography-neutral)
- FAQ page (8 questions, accordion format)
- Shipping policy
- Returns & exchanges policy
- Privacy policy
- Contact page
- 4 product listings with variant configuration

### Home Page
- 3-column product grid with square images
- Enlarged product card titles (22px) — applied via inline
  styles in section Liquid to override theme specificity
- Section subtitle and improved heading typography
- Hover image swap (secondary/open-box image) restored —
  inline CSS in `sections/featured-collection.liquid` `<style>`
  block; no media query restriction, `opacity: 1 !important`
  on reveal rule to bypass Dawn's desktop-only gate

### Product Page
- Improved media gallery layout
- Better product description typography and layout
- Colour variant buttons styled with actual colour swatches
  (via `data-color-value` attribute and inline styles in
  `northforge-custom.css`)

### Cart Page
- "Continue shopping" button styled as green pill shape
- Removed border/box-shadow from button

### Footer
- Navigable sitemap tree
- Catalogue link points to `/collections/all`
- Catalogue group open by default; Pages and Policies
  groups collapsed by default (user clicks to expand)
- PayPal removed from payment icons

### FAQ Page (`fix-faq-accordion.js`)
- Full accordion using native `<details>`/`<summary>` HTML
- Styles embedded directly in page HTML via `<style>` tag
- Bold question text (`font-weight: 700`, `2rem`)
- Left-side `▶` arrow that rotates 90° downward when open
- Answer text: `1.8rem`, line-height 1.75
- Footer note: `1.6rem`
- Borders between items using `rgba(18,18,18,0.12)`

### About Page
- Green pill-shaped CTA button

### Navigation (Main Menu)
- About (links to `/pages/about-northforge-labs`)
- FAQ (links to `/pages/frequently-asked-questions`)

---

## In Progress / Pending

- Product photography reshoot (lightbox arriving — reshoot
  brief prepared, session planned)
- PayPal → Wise weekly auto-transfer (waiting on
  micro-deposit verification in Wise)
- Shipping supplies order (label printer, scale, boxes)
- H2D test print data capture (print time + filament per bundle)
- Tray configuration system: Standard Bundle and Commander Bundle
  still need variant restructure + tray count tags + correct base
  pricing (tray counts and prices TBD)

---

## Admin Scripts (`northforge-upload/`)

| Script | Purpose |
|---|---|
| `fix-faq-accordion.js` | Rebuilds FAQ page with accordion HTML + embedded styles |
| `fetch-faq-page.js` | Fetches and prints current FAQ page HTML |
| `fix-about-page.js` | Updates About page content |
| `fix-about-button.js` | Updates About page CTA button |
| `fix-description.js` | Updates a product description |
| `list-products.js` | Lists all products with IDs |
| `upload-images.js` | Uploads images to Shopify |
| `add-battle-variants.js` | Adds variants to the Battle Bundle product |
| `fix-battle-images.js` | Fixes Battle Bundle product images |
| `fix-battle-inventory.js` | Fixes inventory on Battle Bundle variants |
| `fix-bundle-price.js` | Updates bundle pricing |
| `create-standard-bundle.js` | Creates a standard bundle product |
| `reassign-variants.js` | Reassigns variants between products |
| `fix-skirmish-variants.js` | Initial Skirmish tray/variant restructure |
| `fix-skirmish-colors.js` | Restored Purple + Black to Skirmish Bundle |
| `fix-battle-variants.js` | Battle Bundle tray/variant setup + tags both bundles |
| `fix-battle-price.js` | Sets Battle Bundle base price to $125 CAD |

---

## Store Details

- **Shop domain:** `northforge-labs.myshopify.com`
- **Currency:** CAD displayed with currency code enabled
- **Shipping:** Customer-paid, calculated at checkout via
  netParcel. Ships Canada, USA, international.
- **Payment:** Shopify Payments (Visa, Mastercard, Amex,
  Shop Pay, Apple Pay, Google Pay) + PayPal
- **Returns:** No returns on made-to-order items; exchanges
  within 7 days for defective/damaged items with photo proof

---

## Key File Locations

```
northforge/                        ← theme repo (git)
├── assets/
│   └── northforge-custom.css      ← custom styles (cart page,
│                                     colour swatches, FAQ base)
├── layout/
│   └── theme.liquid               ← global layout, font/CSS loading
├── sections/
│   └── main-page.liquid           ← renders Shopify page content
│                                     via {{ page.content }}
└── templates/
    └── page.json                  ← default page template

northforge-upload/                 ← Admin API scripts (not in git)
└── fix-faq-accordion.js           ← most recently edited
```

---

## Business Context (for AI advisors)

- Sole proprietorship, Kitchener/Waterloo region, Canada
- Single Bambu Lab H2D printer, ~25 unit/month capacity
- Made-to-order model, no pre-built inventory
- Stage 1 of 4 — validation phase, target $2k-$4k/month revenue
- Capital discipline: $10k operating reserve, 50% reinvestment
- Geography-neutral copy throughout (no Canada references)
- War-Ganizer Workshop SKU expansion deferred to post-launch
- No traffic being directed to store yet — pre-launch phase
