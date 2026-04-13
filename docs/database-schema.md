# ThreadIQ Database Schema

Designed for Supabase (Postgres). All tables use UUID primary keys and timestamps.

## Tables Overview

| Table | Purpose |
|-------|---------|
| `brands` | Brand records (Free People, FP Movement, vintage labels) |
| `items` | Canonical item records |
| `item_variants` | Size, color, print, wash variations |
| `market_listings` | Active listing snapshots from tracked sources |
| `market_sales` | Sold comp records |
| `pricing_snapshots` | Aggregated pricing metrics by date |
| `rarity_scores` | Calculated scarcity data |
| `availability_status` | Current retail/resale availability |
| `seller_calc_rules` | Platform fee assumptions and pricing formulas |
| `images` | Item images and alternate views |
| `tags` | Keywords for matching and search |

---

## brands

```sql
CREATE TABLE brands (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  parent_brand TEXT,
  brand_type TEXT CHECK (brand_type IN ('contemporary', 'vintage_label', 'house_brand', 'designer')),
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
```

Examples: Free People, FP Movement, We The Free, Levi's (vintage), Harley-Davidson (vintage)

---

## items

```sql
CREATE TABLE items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  brand_id UUID REFERENCES brands(id),
  canonical_name TEXT NOT NULL,
  line_name TEXT,
  category TEXT NOT NULL,
  subcategory TEXT,
  style_code TEXT,
  original_retail_price NUMERIC(10,2),
  release_year INTEGER,
  era TEXT,
  fabric TEXT,
  color_family TEXT,
  pattern TEXT,
  silhouette TEXT,
  description_normalized TEXT,
  item_type TEXT CHECK (item_type IN ('free_people', 'vintage', 'other')),
  status TEXT CHECK (status IN ('active', 'archived', 'discontinued')) DEFAULT 'active',
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_items_brand ON items(brand_id);
CREATE INDEX idx_items_category ON items(category);
CREATE INDEX idx_items_canonical_name ON items USING gin(to_tsvector('english', canonical_name));
```

---

## item_variants

```sql
CREATE TABLE item_variants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id) ON DELETE CASCADE,
  size TEXT,
  color TEXT,
  print_name TEXT,
  wash TEXT,
  label_variant TEXT,
  sku TEXT,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_variants_item ON item_variants(item_id);
```

---

## market_listings

```sql
CREATE TABLE market_listings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id),
  source TEXT NOT NULL,
  source_listing_id TEXT,
  listing_title TEXT NOT NULL,
  listing_price NUMERIC(10,2) NOT NULL,
  shipping_price NUMERIC(10,2),
  condition_grade TEXT,
  size TEXT,
  color TEXT,
  seller_name TEXT,
  listing_url TEXT,
  first_seen_at TIMESTAMPTZ DEFAULT now(),
  last_seen_at TIMESTAMPTZ DEFAULT now(),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_listings_item ON market_listings(item_id);
CREATE INDEX idx_listings_source ON market_listings(source);
CREATE INDEX idx_listings_active ON market_listings(is_active);
```

Sources: ebay, poshmark, depop, mercari, nuuly_resale, freepeople_com

---

## market_sales

```sql
CREATE TABLE market_sales (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id),
  source TEXT NOT NULL,
  sale_price NUMERIC(10,2) NOT NULL,
  sale_date DATE,
  condition_grade TEXT,
  size TEXT,
  color TEXT,
  seller_name TEXT,
  sale_url TEXT,
  listing_title TEXT,
  confidence TEXT CHECK (confidence IN ('high', 'medium', 'low')) DEFAULT 'medium',
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_sales_item ON market_sales(item_id);
CREATE INDEX idx_sales_date ON market_sales(sale_date);
CREATE INDEX idx_sales_source ON market_sales(source);
```

---

## pricing_snapshots

```sql
CREATE TABLE pricing_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id),
  snapshot_date DATE NOT NULL,
  comp_count INTEGER,
  price_low NUMERIC(10,2),
  price_median NUMERIC(10,2),
  price_high NUMERIC(10,2),
  price_mean NUMERIC(10,2),
  active_listing_count INTEGER,
  pricing_confidence TEXT CHECK (pricing_confidence IN ('high', 'medium', 'low')),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_snapshots_item_date ON pricing_snapshots(item_id, snapshot_date);
```

---

## rarity_scores

```sql
CREATE TABLE rarity_scores (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id),
  score_date DATE NOT NULL,
  active_count INTEGER,
  total_historical_comps INTEGER,
  rarity_tier TEXT CHECK (rarity_tier IN ('common', 'uncommon', 'scarce', 'very_scarce')),
  demand_score NUMERIC(3,2),
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_rarity_item ON rarity_scores(item_id);
```

Scoring rules:
- Very Scarce: 0-2 active, <5 historical comps
- Scarce: 3-5 active, 5-15 comps
- Uncommon: 6-15 active, 15-50 comps
- Common: 15+ active, 50+ comps

---

## availability_status

```sql
CREATE TABLE availability_status (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id),
  status TEXT CHECK (status IN ('retail_active', 'resale_affiliated', 'resale_secondary', 'no_listings')),
  checked_at TIMESTAMPTZ DEFAULT now(),
  source TEXT,
  source_url TEXT,
  notes TEXT
);

CREATE INDEX idx_availability_item ON availability_status(item_id);
```

---

## seller_calc_rules

```sql
CREATE TABLE seller_calc_rules (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  platform TEXT NOT NULL,
  category TEXT,
  commission_rate NUMERIC(5,4) NOT NULL,
  processing_rate NUMERIC(5,4),
  transaction_fee NUMERIC(10,2),
  notes TEXT,
  effective_date DATE,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

Default Whatnot row:
- platform: whatnot
- category: all_other
- commission_rate: 0.0800
- processing_rate: 0.0290
- transaction_fee: 0.30

---

## images

```sql
CREATE TABLE images (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id) ON DELETE CASCADE,
  image_url TEXT NOT NULL,
  image_type TEXT CHECK (image_type IN ('primary', 'alternate', 'tag', 'label', 'detail')),
  source TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_images_item ON images(item_id);
```

---

## tags

```sql
CREATE TABLE tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id) ON DELETE CASCADE,
  tag TEXT NOT NULL,
  tag_type TEXT CHECK (tag_type IN ('keyword', 'style', 'era', 'fabric', 'graphic', 'label', 'other')),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_tags_item ON tags(item_id);
CREATE INDEX idx_tags_text ON tags USING gin(to_tsvector('english', tag));
```

---

## Whatnot Net Price Calculator (SQL Function)

```sql
CREATE OR REPLACE FUNCTION calculate_whatnot_net(
  sale_price NUMERIC,
  platform TEXT DEFAULT 'whatnot',
  category TEXT DEFAULT 'all_other'
) RETURNS TABLE (
  gross_price NUMERIC,
  commission NUMERIC,
  processing_fee NUMERIC,
  transaction_fee NUMERIC,
  total_fees NUMERIC,
  seller_net NUMERIC
) AS $$
BEGIN
  RETURN QUERY
  SELECT
    sale_price AS gross_price,
    ROUND(sale_price * r.commission_rate, 2) AS commission,
    ROUND(sale_price * r.processing_rate, 2) AS processing_fee,
    r.transaction_fee,
    ROUND(sale_price * r.commission_rate + sale_price * r.processing_rate + r.transaction_fee, 2) AS total_fees,
    ROUND(sale_price - (sale_price * r.commission_rate + sale_price * r.processing_rate + r.transaction_fee), 2) AS seller_net
  FROM seller_calc_rules r
  WHERE r.platform = calculate_whatnot_net.platform
    AND r.category = calculate_whatnot_net.category
  ORDER BY r.effective_date DESC NULLS LAST
  LIMIT 1;
END;
$$ LANGUAGE plpgsql;
```

Usage: `SELECT * FROM calculate_whatnot_net(65.00);`
Returns: gross $65.00, commission $5.20, processing $1.89, txn fee $0.30, total fees $7.39, net $57.62
