# Crocs Technical Prep — SQL & Snowflake
*Role: Sr. BI Analyst | Round 2 Technical*

---

## SQL — Core Concepts to Know Cold

### Window Functions
The most commonly tested advanced SQL topic. Know all of these without hesitation.

```sql
-- RANK vs DENSE_RANK vs ROW_NUMBER
-- RANK skips numbers after ties. DENSE_RANK doesn't. ROW_NUMBER is always unique.
SELECT
    product_name,
    revenue,
    RANK()         OVER (PARTITION BY brand ORDER BY revenue DESC) AS rank_with_gaps,
    DENSE_RANK()   OVER (PARTITION BY brand ORDER BY revenue DESC) AS rank_no_gaps,
    ROW_NUMBER()   OVER (PARTITION BY brand ORDER BY revenue DESC) AS unique_row
FROM sales;

-- LAG / LEAD — compare current row to previous/next
SELECT
    month,
    revenue,
    LAG(revenue, 1)  OVER (PARTITION BY brand ORDER BY month) AS prior_month_rev,
    revenue - LAG(revenue, 1) OVER (PARTITION BY brand ORDER BY month) AS mom_change
FROM monthly_sales;

-- Running total
SELECT
    sale_date,
    daily_revenue,
    SUM(daily_revenue) OVER (PARTITION BY brand ORDER BY sale_date
                              ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ytd_revenue
FROM daily_sales;

-- NTILE — bucket into quartiles
SELECT
    store_id,
    revenue,
    NTILE(4) OVER (ORDER BY revenue DESC) AS revenue_quartile
FROM store_performance;
```

### CTEs vs Subqueries
Prefer CTEs for readability and step-by-step logic. Know when each is appropriate.

```sql
-- CTE — readable, reusable within the query
WITH brand_totals AS (
    SELECT brand, SUM(revenue) AS total_rev
    FROM sales
    GROUP BY brand
),
brand_share AS (
    SELECT
        brand,
        total_rev,
        total_rev / SUM(total_rev) OVER () AS revenue_share
    FROM brand_totals
)
SELECT * FROM brand_share ORDER BY revenue_share DESC;
```

### Retail-Specific Query Patterns (Crocs-Relevant)

**Sell-through rate:**
```sql
SELECT
    product_id,
    product_name,
    units_sold,
    units_available,
    ROUND(units_sold::FLOAT / NULLIF(units_available, 0) * 100, 1) AS sell_through_pct
FROM inventory
ORDER BY sell_through_pct DESC;
```

**YoY revenue comparison by brand and channel:**
```sql
WITH current_year AS (
    SELECT brand, channel, SUM(revenue) AS cy_revenue
    FROM sales
    WHERE YEAR(sale_date) = 2026
    GROUP BY brand, channel
),
prior_year AS (
    SELECT brand, channel, SUM(revenue) AS py_revenue
    FROM sales
    WHERE YEAR(sale_date) = 2025
    GROUP BY brand, channel
)
SELECT
    c.brand,
    c.channel,
    c.cy_revenue,
    p.py_revenue,
    ROUND((c.cy_revenue - p.py_revenue) / NULLIF(p.py_revenue, 0) * 100, 1) AS yoy_growth_pct
FROM current_year c
LEFT JOIN prior_year p ON c.brand = p.brand AND c.channel = p.channel
ORDER BY brand, yoy_growth_pct;
```

**Top N products per brand (common interview question):**
```sql
WITH ranked AS (
    SELECT
        brand,
        product_name,
        SUM(revenue) AS total_revenue,
        ROW_NUMBER() OVER (PARTITION BY brand ORDER BY SUM(revenue) DESC) AS rn
    FROM sales
    GROUP BY brand, product_name
)
SELECT brand, product_name, total_revenue
FROM ranked
WHERE rn <= 5;
```

**Cohort retention (may come up if they discuss DTC / repeat purchase):**
```sql
WITH first_purchase AS (
    SELECT customer_id, MIN(DATE_TRUNC('month', order_date)) AS cohort_month
    FROM orders
    GROUP BY customer_id
),
activity AS (
    SELECT
        o.customer_id,
        f.cohort_month,
        DATE_TRUNC('month', o.order_date) AS activity_month,
        DATEDIFF('month', f.cohort_month, DATE_TRUNC('month', o.order_date)) AS months_since_first
    FROM orders o
    JOIN first_purchase f ON o.customer_id = f.customer_id
)
SELECT
    cohort_month,
    months_since_first,
    COUNT(DISTINCT customer_id) AS retained_customers
FROM activity
GROUP BY cohort_month, months_since_first
ORDER BY cohort_month, months_since_first;
```

### Performance Optimization (SQL)
- **Filter early** — push WHERE clauses as far upstream as possible (especially in CTEs)
- **Avoid SELECT \*** — columns you don't need still get loaded
- **NULLIF for safe division** — never divide without protecting against zero
- **EXISTS vs IN** — EXISTS short-circuits; prefer it for large subqueries checking existence
- **Avoid functions on indexed columns in WHERE** — `WHERE YEAR(date_col) = 2026` can't use an index; `WHERE date_col BETWEEN '2026-01-01' AND '2026-12-31'` can

---

## Snowflake — Key Concepts

### Virtual Warehouses
- Warehouses are compute; storage is separate — you scale them independently
- **Auto-suspend / auto-resume** — set suspend to 60–120 seconds to avoid idle cost; resume is near-instant
- **Multi-cluster warehouses** — scale out for concurrent users (not just bigger queries); important for shared BI platforms where 20 people hit the warehouse at once
- **Warehouse sizing:** XS for dev/light queries, S–M for typical BI loads, L+ for large transforms

```sql
-- Suspend/resume manually
ALTER WAREHOUSE my_wh SUSPEND;
ALTER WAREHOUSE my_wh RESUME;

-- Check what queries are running
SELECT * FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY()) LIMIT 20;
```

### Clustering Keys
Used when large tables are queried repeatedly on the same filter columns (e.g., `brand`, `sale_date`). Snowflake uses micro-partitions; clustering ensures relevant data is co-located.

```sql
ALTER TABLE sales CLUSTER BY (brand, sale_date);

-- Check clustering health
SELECT SYSTEM$CLUSTERING_INFORMATION('sales', '(brand, sale_date)');
```

Don't cluster small tables — it adds overhead without benefit.

### Zero-Copy Cloning
Creates an instant snapshot without copying data — useful for dev/test environments.

```sql
CREATE TABLE sales_dev CLONE sales;
CREATE DATABASE crocs_dev CLONE crocs_prod;
```

### Time Travel
Query data as it existed at a point in the past. Default retention is 1 day (Enterprise: up to 90 days).

```sql
-- As of a timestamp
SELECT * FROM sales AT (TIMESTAMP => '2026-04-30 09:00:00'::TIMESTAMP);

-- Undo a bad DELETE (restore from before the statement)
CREATE TABLE sales_restored CLONE sales BEFORE (STATEMENT => '<query_id>');
```

### Views vs Materialized Views
- **View:** No storage, re-executes on every query. Use for simple logical abstraction.
- **Materialized view:** Stores pre-computed results, auto-refreshes incrementally. Use for expensive aggregations queried frequently.

```sql
CREATE MATERIALIZED VIEW mv_brand_daily_revenue AS
SELECT brand, DATE_TRUNC('day', sale_date) AS day, SUM(revenue) AS daily_rev
FROM sales
GROUP BY 1, 2;
```

### Role-Based Access Control (RBAC)
Snowflake uses a role hierarchy. Key roles: ACCOUNTADMIN > SYSADMIN > custom roles.

```sql
-- Grant access pattern
GRANT USAGE ON DATABASE crocs_db TO ROLE bi_analyst_role;
GRANT USAGE ON SCHEMA crocs_db.analytics TO ROLE bi_analyst_role;
GRANT SELECT ON ALL TABLES IN SCHEMA crocs_db.analytics TO ROLE bi_analyst_role;
```

### Query Performance Tips
- **Result cache** — identical queries within 24 hours return cached results instantly (no warehouse needed)
- **Query profile** — use the Snowflake UI query profile to identify expensive nodes (TableScan, Aggregate)
- **Avoid DISTINCT where possible** — GROUP BY often performs better
- **Partition pruning** — filter on columns that align with how data is partitioned (usually date fields)

---

## Practice Problems — Do These Out Loud

1. *"Write a query that returns each brand's revenue as a % of total enterprise revenue for each month of 2026."*

2. *"We have a sales table with columns: order_id, customer_id, brand, channel, order_date, revenue. Write a query that identifies customers who purchased from both the Crocs brand and HEYDUDE within the same calendar year."*

3. *"Find the month-over-month revenue change for each brand/channel combination, and flag any month where the decline was greater than 10%."*

4. *"You have an inventory table and a sales table. Write a query that returns products where sell-through rate is below 30% and inventory has been sitting for more than 90 days."*

5. *"In Snowflake, a dashboard is running slowly. Walk me through how you'd diagnose and fix it."*
   - Check Query Profile for bottlenecks
   - Look at scan percentage (are micro-partitions being pruned?)
   - Check if warehouse is under-sized vs concurrent user issue
   - Consider clustering key or materialized view
   - Check if result cache is being bypassed (e.g., dynamic date filters)
