# ThreadIQ Build Plan

## Phase 1: Foundation (Weeks 1-3)

### Goal
Free People items searchable with basic pricing data.

### Tasks
- [ ] Set up Supabase project and create database tables from schema
- [ ] Seed `brands` table with Free People sub-brands (Free People, FP Movement, We The Free, Intimately FP)
- [ ] Seed `seller_calc_rules` with Whatnot fee structure
- [ ] Load 50-100 Free People items manually from sample CSV
- [ ] Build basic search API (Supabase edge function or direct query)
- [ ] Build static HTML/JS front end with search bar and results list
- [ ] Deploy concept app to GitHub Pages

### Deliverables
- Working search across seeded Free People items
- Item detail page with basic pricing info
- Whatnot net calculator (client-side JS)

---

## Phase 2: Comp Engine (Weeks 4-6)

### Goal
eBay sold listing ingestion via TinyFish, pricing calculations working.

### Tasks
- [ ] Set up TinyFish eBay sold listing workflow (see tinyfish-ebay-workflow.md)
- [ ] Build ingestion script to parse TinyFish output into `market_sales` table
- [ ] Build title normalization logic to match raw listings to canonical items
- [ ] Calculate and store `pricing_snapshots` (low/median/high/confidence)
- [ ] Calculate and store `rarity_scores` from listing counts
- [ ] Display comp data on item detail pages
- [ ] Add Whatnot pricing recommendations (start bid, BIN, net)

### Deliverables
- Automated or semi-automated eBay comp collection
- Pricing engine with confidence scores
- Scarcity estimates on item profiles

---

## Phase 3: Vintage Categories (Weeks 7-10)

### Goal
Expand beyond Free People to vintage clothing with attribute-based matching.

### Tasks
- [ ] Add vintage brand records (Harley-Davidson, Levi's, band tees, etc.)
- [ ] Build attribute-based search (era, label type, fabric, graphic theme, construction)
- [ ] Add vintage-specific scarcity signals (single-stitch, union label, made-in-USA, tag style)
- [ ] Expand TinyFish workflows for vintage search terms
- [ ] Build fuzzy matching for vintage items (no exact SKU)
- [ ] Add "similar comps" feature for items without exact matches

### Deliverables
- Vintage items searchable by attributes
- Comp ranges for common vintage categories
- Wider pricing bands with appropriate low-confidence labels

---

## Phase 4: Smart Features (Weeks 11-14)

### Goal
Image recognition, trend detection, and live show mode.

### Tasks
- [ ] Add image upload for tag/label photo identification
- [ ] Build "live show mode" with large-format pricing cards
- [ ] Add trending items dashboard (biggest price moves, new comps)
- [ ] Add saved searches and alerts for price changes
- [ ] Build price history charts per item
- [ ] Add multi-platform fee calculator (eBay, Poshmark, Depop alongside Whatnot)

### Deliverables
- Photo-based item lookup
- Streaming-friendly pricing display
- Price trend visualization

---

## Phase 5: Scale & Distribution (Weeks 15+)

### Goal
Browser extension, mobile access, community features.

### Tasks
- [ ] Chrome extension for quick lookup while browsing eBay/thrift sites
- [ ] Mobile-responsive PWA for sourcing on the go
- [ ] User accounts with saved inventory and pricing history
- [ ] Community comp submissions (crowdsourced pricing data)
- [ ] API access for power users and integration with listing tools

### Deliverables
- Browser extension MVP
- Mobile-friendly interface
- User accounts and saved data

---

## Tech Stack

| Layer | Tool | Notes |
|-------|------|-------|
| Database | Supabase (Postgres) | Free tier to start, Row Level Security for future auth |
| Backend | Supabase Edge Functions | Serverless, no infra to manage |
| Frontend | Static HTML/JS or Next.js | GitHub Pages for MVP, Vercel for production |
| Data Ingestion | TinyFish + Python scripts | eBay sold listings, Free People reference data |
| Search | Postgres full-text search | GIN indexes on item names and tags |
| Auth | Supabase Auth | Add in Phase 5 when user accounts needed |
| Image Recognition | OpenAI Vision or Google Lens API | Phase 4 |
| Hosting | GitHub Pages -> Vercel | Free to start |

---

## Data Seeding Priority

### Free People (Phase 1)
Focus on the most commonly resold styles first:
1. Dresses (Adella, FP One, etc.)
2. Tops and thermals
3. Intimately FP bralettes and bodysuits
4. Sweaters and cardigans
5. Jackets

### Vintage (Phase 3)
Focus on highest-volume Whatnot categories:
1. Band/music tees
2. Harley-Davidson
3. Sports/college
4. Western/cowboy
5. 90s streetwear

---

## Success Metrics

- **Phase 1**: 100 items seeded, search returns results in <500ms
- **Phase 2**: 500+ sold comps ingested, pricing confidence on 50%+ of items
- **Phase 3**: 200+ vintage items, attribute search working
- **Phase 4**: Image lookup functional, live show mode usable during stream
- **Phase 5**: 10+ active users, browser extension in Chrome Web Store
