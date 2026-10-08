# kroger-products-etl

A Databricks Serverless ETL pipeline that extracts product data from the Kroger API, deduplicates, transforms, and builds powerful analytics tables for pricing and product trends. Everything is managed in Unity Catalog Delta tables.

## Architecture
```
Kroger API → Bronze (raw) → Silver (cleaned, deduped) → Gold (analytics)
```

## Kroger_Products_ETL: What It Does
- Authenticates via Databricks secrets (OAuth2)
- Extracts products across core grocery terms (meat, seafood, grains, dairy, fruit, veg, legumes, spice, condiment, snacks)
- Deduplicates by product ID and UPC
- Flattens JSON to tabular rows
- Creates bronze table with all raw products
- Builds silver table: deduplication per product/day, price cast and product name column
- Creates gold table: price reduction % analytics, daily trends
- Persists top 10 price reduction deals to gold table and `kroger.daily_best_deals`
- Finds cheapest item each run

## Main Data Tables
| Table | Layer | Description |
|-------|-------|---------------------------------------------------------|
| kroger.products | Bronze | Raw extracted product data |
| kroger.products_silver | Silver | Cleaned products; earliest per-date ingestion |
| kroger.products_gold | Gold | Price reduction %, trend analytics |
| kroger.daily_best_deals | Gold | Daily top 10 price reduction deals |

## Gold Table Key Columns
- `description_size`: Unique product descriptor
- `date`: Day granularity
- `price_of_day`: Price for the day
- `price_reduction_%`: Drop from previous price

## How to Run
1. Store your Kroger API credentials (`client_id`, `client_secret`, `location_id`) in the Databricks secret scope `productsetl`
2. Run the `Kroger_Products_ETL` notebook; tables are created/updated automatically
3. Use `kroger.products_gold` and `kroger.daily_best_deals` for dashboards or queries: daily best deals, price drops, cheapest-item analysis

## Tech Stack
- Databricks Serverless Compute
- Unity Catalog Delta Tables
- PySpark/Spark SQL
- Kroger API

## Example Dashboard Ideas
- Price drop leaderboard by day
- Cheapest item finder
- Daily trend charts by product or category
- Brand/category share (can be extended)
