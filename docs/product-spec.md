# ThreadIQ Product Spec

## Product Overview

ThreadIQ is a resale intelligence web app for Free People and vintage clothing sellers on Whatnot and other resale platforms. It answers four questions fast: What is this item? How common is it? What is it selling for? What should I charge on Whatnot?

## Target Users

### Primary: Whatnot Live Sellers
Sellers who need to price quickly during sourcing, show prep, or live selling. They deal in Free People, boho, and vintage clothing categories.

### Secondary: Multi-Platform Resellers
Sellers on eBay, Poshmark, Depop, or Nuuly Resale who want comps and scarcity context before listing.

### Tertiary: Boutique Vintage Dealers
Dealers who care about era, label, construction details, and demand signals rather than SKU-level matching.

## Core Modules

### 1. Item Search & Matching
- Search by brand, style name, category, fabric, print, color, size, keywords, tag text
- Free People matching: prioritize exact style names and normalized variants
- Vintage matching: attribute-based (era, label type, construction, fabric, graphic theme)
- Fuzzy matching for imperfect seller input ("FP floral midi" -> "Free People Floral Midi Dress")

### 2. Item Profile Page
- Canonical item title
- Brand and sub-line (Free People, FP Movement, We The Free, etc.)
- Category and silhouette
- Original retail price (when available)
- Active listing count across tracked sources
- Sold comp range (low / median / high)
- Estimated rarity score
- Availability status: retail active, resale active, archived, no signal

### 3. Pricing Assistant
Outputs:
- Suggested Whatnot auction start price
- Safe reserve floor
- Recommended BIN (buy it now) price
- Expected seller net after fees
- Pricing confidence score (based on comp count, spread, recency)

Whatnot fee model (US, clothing/vintage = "all other items"):
- 8% commission on final sale price
- 2.9% payment processing on total order value
- $0.30 transaction fee per order

### 4. Scarcity Meter
Rarity tiers: Common / Uncommon / Scarce / Very Scarce

Based on:
- Active listing count across sources
- Sold comp frequency
- Style age and release window
- Current retail/resale availability
- For vintage: era, label type, single-stitch vs double-stitch, made-in country

NOTE: Exact production quantities are generally unavailable for both Free People and vintage. The system uses market frequency as a proxy, not published unit counts.

### 5. Availability Tracker
Statuses:
- Available at retail (brand site, department stores)
- Available on affiliated resale (Nuuly Resale)
- Available only on secondary resale (eBay, Poshmark, Depop, etc.)
- No active listings found

## Scoring Logic

### Pricing Confidence
- High: 10+ comps, tight spread (<30% variance), recent sales (last 90 days)
- Medium: 5-9 comps, moderate spread, sales within 6 months
- Low: <5 comps, wide spread, or old/stale data

### Demand Score
- Based on ratio of sold count to active count
- High demand: items sell faster than they get listed
- Low demand: many active listings, few recent sales

### Scarcity Score
- Very Scarce: 0-2 active listings, <5 total historical comps
- Scarce: 3-5 active, 5-15 comps
- Uncommon: 6-15 active, 15-50 comps
- Common: 15+ active, 50+ comps

## User Flows

### Flow 1: Free People Lookup
1. Seller searches "Free People Adella Slip Dress"
2. App returns canonical item with variants (colors, sizes)
3. Profile shows: $148 retail, $45-$85 resale range, 23 active listings, scarcity: Common
4. Whatnot calculator: Start at $25, BIN at $65, estimated net $56.43

### Flow 2: Vintage Lookup
1. Seller enters "single stitch Harley wolf tee 90s"
2. App matches by attributes (not SKU)
3. Profile shows: wide range $40-$180, 4 comps, scarcity: Scarce, confidence: Low
4. Whatnot calculator: Conservative start $30, fair BIN $90, premium BIN $140

### Flow 3: Quick Price During Live Show
1. Seller opens "live show mode" (large-format pricing cards)
2. Searches item by name or tag photo
3. Sees large-format suggested price and net payout
4. Sets price on stream within seconds

## Interface Structure

| Screen | Purpose |
|--------|---------|
| Dashboard | Recent searches, trending items, pricing alerts |
| Search Results | Filtered results with match confidence |
| Item Detail | Full profile, comps, pricing, scarcity |
| Seller Calculator | Whatnot net estimator with fee breakdown |
| Live Show Mode | Large-format pricing cards for streaming |
| Admin / Data Review | Approve matches, merge duplicates, adjust records |

## Data Sources

### eBay Sold Listings (Primary Comp Source)
Collected via TinyFish browser automation. See tinyfish-ebay-workflow.md.
Fields captured: title, sold price, sale date, condition, seller, listing URL.

### Free People / Nuuly Resale (Retail Reference)
Used for original retail pricing, current availability, and item identification.
Nuuly Resale exposes item names, sale prices, and original prices on category pages.

### Manual Entry
For verified rare pieces, auction results, and corrections.

## Key Design Decisions

1. **Scarcity over quantity**: We use market frequency, not production counts, because exact manufacturing data is unavailable for both Free People and vintage.
2. **Net pricing over gross**: Whatnot sellers care about take-home, so all pricing includes fee math.
3. **Confidence scores over single prices**: Resale markets have high variance. A range with confidence is more honest and useful than a single number.
4. **Free People first**: FP has structured, recurring styles that are easier to normalize. Vintage comes in Phase 3.
5. **Attribute matching for vintage**: One-of-one items need era/label/fabric/graphic matching, not SKU lookup.
