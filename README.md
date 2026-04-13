# ThreadIQ

**Resale intelligence database for Free People & vintage clothing — built for Whatnot sellers.**

ThreadIQ helps live sellers identify items, estimate fair pricing, track market availability, and calculate net payout after platform fees. Built for speed during sourcing, show prep, and live selling.

## What It Does

- **Item Search** — Find Free People styles and vintage clothing by name, tag text, category, fabric, era, or keywords
- **Pricing Engine** — Low / median / high sold comp ranges with confidence scoring
- **Whatnot Calculator** — Suggested start bid, BIN price, and estimated seller net after 8% commission + processing fees
- **Scarcity Meter** — Market frequency-based rarity estimates (common / uncommon / scarce / very scarce)
- **Availability Tracker** — Retail active, resale active, archived, or no current listings

## Why This Exists

Whatnot sellers need to price fast and price right. Free People has a massive resale footprint with recurring styles, but sellers waste time manually searching comps across platforms. Vintage is even harder — one-of-one items with no SKU and wide price variance.

ThreadIQ solves this by centralizing comp data, normalizing messy listing titles into canonical items, and outputting actionable pricing with platform-specific fee math.

## Data Model

The database tracks:
- **Canonical items** — Normalized brand, style, category, fabric, era, original retail price
- **Market listings** — Active listings with source, price, condition, size
- **Sold comps** — Historical sale prices with date, condition, confidence
- **Rarity scores** — Calculated from market frequency and availability signals
- **Pricing snapshots** — Aggregated metrics over time for trend tracking

See [docs/database-schema.md](docs/database-schema.md) for the full schema.

## Data Collection

ThreadIQ uses a multi-source ingestion approach:
- **eBay sold listings** via TinyFish browser automation (see [docs/tinyfish-ebay-workflow.md](docs/tinyfish-ebay-workflow.md))
- **Free People / Nuuly Resale** reference data for retail pricing and availability
- **Manual entry** for verified comps and rare vintage pieces

See [docs/build-plan.md](docs/build-plan.md) for the full roadmap.

## Project Structure

```
threadiq/
├── README.md
├── docs/
│   ├── product-spec.md
│   ├── database-schema.md
│   ├── build-plan.md
│   └── tinyfish-ebay-workflow.md
├── app/
│   └── index.html
├── scripts/
│   └── sample-items.csv
└── roadmap.md
```

## Tech Stack (Planned)

- **Frontend**: Next.js or static HTML/JS
- **Database**: Supabase (Postgres)
- **Auth**: Supabase Auth or Clerk
- **Search**: Postgres full-text search, vector search later
- **Data Ingestion**: Python scripts + TinyFish browser automation
- **Hosting**: GitHub Pages (concept) → Vercel (production)

## Roadmap

| Phase | Focus |
|-------|-------|
| 1 | Free People items, manual + semi-auto data seeding |
| 2 | Whatnot calculator, item profiles, saved comps |
| 3 | Vintage categories with attribute-based matching |
| 4 | Image recognition from photos or tag screenshots |
| 5 | Browser extension or mobile quick-lookup for sourcing |

## Getting Started

1. Clone the repo
2. Open `app/index.html` for the concept UI
3. Review `docs/` for specs and schema
4. Load `scripts/sample-items.csv` into your database

## License

MIT
