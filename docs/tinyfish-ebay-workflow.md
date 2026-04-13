# TinyFish eBay Sold Listings Workflow

This document describes how to use TinyFish browser automation to collect eBay sold listing data for ThreadIQ's pricing engine.

## Overview

TinyFish is a browser automation tool that can navigate dynamic websites and extract structured data. We use it to search eBay's completed/sold listings and capture pricing comps that feed into our `market_sales` table.

## Why eBay Sold Listings

eBay sold listings are the gold standard for resale pricing because:
- They reflect actual buyer behavior, not seller hopes
- High volume across Free People and vintage categories
- Condition, size, and date are usually visible
- Completed listings stay accessible for ~90 days

## Workflow Architecture

```
TinyFish Agent
    |
    v
eBay Sold Search (browser automation)
    |
    v
Raw Listing Data (JSON/CSV output)
    |
    v
Python Parser Script
    |
    v
Title Normalization + Item Matching
    |
    v
Supabase market_sales table
    |
    v
Pricing Snapshot Calculation
```

## Step 1: TinyFish Search Task

Configure TinyFish to:
1. Navigate to eBay
2. Enter search query (e.g., "Free People Adella Slip Dress")
3. Apply filters: Sold Items, Completed Listings
4. Optionally filter by category (Women's Clothing)
5. Sort by date (most recent first)
6. Extract data from each result on the page

### Fields to Capture Per Listing

| Field | eBay Location | Maps To |
|-------|--------------|----------|
| Listing title | Item title text | `listing_title` |
| Sold price | Green "Sold" price | `sale_price` |
| Sale date | Date sold text | `sale_date` |
| Condition | Condition label | `condition_grade` |
| Seller name | Seller ID link | `seller_name` |
| Listing URL | Item link href | `sale_url` |
| Shipping cost | Shipping text (if visible) | `notes` |
| Best offer accepted | "or Best Offer" + accepted flag | `notes` |

### Important Notes on eBay Data

- **Best Offer**: Some listings show "or Best Offer" and the actual accepted price may differ from the listed price. Flag these as `confidence: 'low'` unless the accepted amount is visible.
- **Lots/Bundles**: Listings with multiple items should be excluded or flagged. Look for keywords like "lot", "bundle", "set of", "x2", "x3".
- **International**: Filter to US sellers when possible to keep pricing consistent.
- **Free shipping vs paid**: Note shipping cost separately; some sellers build it into the item price.

## Step 2: Output Format

TinyFish should output structured data. Target format:

```json
{
  "search_query": "Free People Adella Slip Dress",
  "search_date": "2026-04-13",
  "results": [
    {
      "listing_title": "Free People Adella Slip Dress Ivory Size Small NWT",
      "sale_price": 62.99,
      "sale_date": "2026-04-10",
      "condition": "New with tags",
      "seller": "thriftqueen2024",
      "url": "https://www.ebay.com/itm/123456789",
      "shipping": "Free shipping",
      "best_offer": false
    },
    {
      "listing_title": "FP Adella Slip Maxi Dress Black M Pre-owned",
      "sale_price": 45.00,
      "sale_date": "2026-04-08",
      "condition": "Pre-owned",
      "seller": "vintagefinds_stl",
      "url": "https://www.ebay.com/itm/987654321",
      "shipping": "$5.99",
      "best_offer": true
    }
  ]
}
```

## Step 3: Python Parser

The parser script handles:
1. **Cleaning**: Remove outliers (prices 3x+ above median or below $5)
2. **Bundle detection**: Skip listings with lot/bundle keywords
3. **Best Offer flagging**: Mark as low confidence
4. **Title normalization**: Strip size, condition, and seller words to extract core item identity
5. **Item matching**: Match normalized title to canonical items in the `items` table
6. **Insert**: Write clean records to `market_sales`

### Title Normalization Examples

| Raw eBay Title | Normalized |
|----------------|------------|
| "Free People Adella Slip Dress Ivory Size Small NWT" | "Free People Adella Slip Dress" |
| "FP Adella Slip Maxi Dress Black M Pre-owned" | "Free People Adella Slip Dress" |
| "FREE PEOPLE intimately adella slip sz L" | "Free People Adella Slip Dress" |
| "Vtg 90s Harley Davidson Eagle Tee XL Single Stitch" | "Harley-Davidson Eagle Graphic Tee" |

Normalization rules:
- Uppercase "FP" -> "Free People"
- Remove size indicators (S, M, L, XL, 0-18, etc.)
- Remove condition words (NWT, NWOT, EUC, pre-owned, used, etc.)
- Remove color words for matching (keep in variant data)
- Remove seller-specific words ("fast ship", "look!", emoji, etc.)
- Collapse whitespace and standardize punctuation

## Step 4: Pricing Snapshot Calculation

After inserting new comps, recalculate the pricing snapshot:

```python
def calculate_snapshot(item_id, lookback_days=90):
    # Get recent comps for this item
    comps = get_comps(item_id, days=lookback_days)
    
    if len(comps) == 0:
        return None
    
    prices = [c.sale_price for c in comps]
    
    snapshot = {
        'item_id': item_id,
        'snapshot_date': today(),
        'comp_count': len(prices),
        'price_low': min(prices),
        'price_median': median(prices),
        'price_high': max(prices),
        'price_mean': mean(prices),
        'active_listing_count': count_active_listings(item_id),
        'pricing_confidence': calculate_confidence(prices)
    }
    
    return snapshot

def calculate_confidence(prices):
    if len(prices) >= 10 and coefficient_of_variation(prices) < 0.30:
        return 'high'
    elif len(prices) >= 5:
        return 'medium'
    else:
        return 'low'
```

## Step 5: Rarity Score Update

After snapshot calculation, update rarity:

```python
def update_rarity(item_id):
    active = count_active_listings(item_id)
    historical = count_total_comps(item_id)
    
    if active <= 2 and historical < 5:
        tier = 'very_scarce'
    elif active <= 5 and historical < 15:
        tier = 'scarce'
    elif active <= 15 and historical < 50:
        tier = 'uncommon'
    else:
        tier = 'common'
    
    return {
        'item_id': item_id,
        'score_date': today(),
        'active_count': active,
        'total_historical_comps': historical,
        'rarity_tier': tier
    }
```

## Search Queries to Run

### Free People (Phase 1)
Run these searches on a weekly or bi-weekly cadence:

```
Free People Adella Slip Dress
Free People FP One Adella Bralette
Free People thermal top
Free People Movement hot shot
Free People maxi dress
We The Free oversized tee
Intimately Free People bodysuit
Free People sweater
Free People jacket
Free People skirt
```

### Vintage (Phase 3)
```
vintage Harley Davidson t-shirt
vintage band tee single stitch
vintage 90s graphic tee
vintage Levi's 501
vintage western shirt
vintage Nike swoosh tee
vintage college university sweatshirt
vintage 80s crop top
```

## Rate Limiting and Best Practices

- Run no more than 10-15 searches per session
- Space searches apart (2-3 minute gaps)
- Rotate search terms rather than hitting the same query repeatedly
- Store raw output before processing (keep a backup)
- Log all runs with timestamp, query, and result count
- Monitor for eBay layout changes that break extraction

## Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| eBay blocks or throttles | Rate limit, rotate queries, use residential IP |
| Layout changes break extraction | Monitor output quality, alert on zero-result runs |
| Messy titles cause bad matches | Title normalization + manual review queue |
| Best Offer hides true price | Flag as low confidence, exclude from median if needed |
| Bundles skew pricing | Keyword filter for lot/bundle/set |
| Stale data | Re-run searches on cadence, expire old snapshots |
