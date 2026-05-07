# Crocs Technical Prep — SQL & Snowflake
*Role: Sr. BI Analyst | Round 2 Technical*  
*Expanded 2026-05-07 — Primary talking points guide*

---

## How to Use This Guide

SQL is your strongest technical lane. Lead with confidence. When you get an open-ended SQL question, think out loud, use CTEs to organize logic, and narrate your reasoning — interviewers at this level want to see *how* you think, not just whether you get the right answer. Reference T-Life scenarios when relevant.

---

## Part 1 — Window Functions (Know Cold)

Window functions are the most tested advanced SQL topic. Know the syntax, the differences, and *when* to use each without hesitation.

### RANK / DENSE_RANK / ROW_NUMBER

```sql
-- RANK: skips numbers after ties (1, 2, 2, 4)
-- DENSE_RANK: no gaps after ties (1, 2, 2, 3)
-- ROW_NUMBER: always unique, arbitrary tiebreak

SELECT
    product_name,
    brand,
    revenue,
    RANK()       OVER (PARTITION BY brand ORDER BY revenue DESC) AS rank_with_gaps,
    DENSE_RANK() OVER (PARTITION BY brand ORDER BY revenue DESC) AS rank_no_gaps,
    ROW_NUMBER() OVER (PARTITION BY brand ORDER BY revenue DESC) AS unique_row
FROM sales;
```

**Interview tip:** "I use ROW_NUMBER when I need exactly one row per group — like deduplication or Top N with no ties. I use DENSE_RANK when I'm building a leaderboard or ranking that consumers will read."

---

### LAG / LEAD — Period-Over-Period Comparisons

```sql
-- Month-over-month revenue change per brand/channel
SELECT
    brand,
    channel,
    month,
    revenue,
    LAG(revenue, 1) OVER (PARTITION BY brand, channel ORDER BY month) AS prior_month_rev,
    revenue - LAG(revenue, 1) OVER (PARTITION BY brand, channel ORDER BY month) AS mom_change,
    ROUND(
        (revenue - LAG(revenue, 1) OVER (PARTITION BY brand, channel ORDER BY month))
        / NULLIF(LAG(revenue, 1) OVER (PARTITION BY brand, channel ORDER BY month), 0) * 100
    , 1) AS mom_pct_change
FROM monthly_sales
ORDER BY brand, channel, month;

-- Flag months where growth reversed (prior was positive, current is negative)
WITH changes AS (
    SELECT
        brand,
        month,
        revenue,
        LAG(revenue) OVER (PARTITION BY brand ORDER BY month) AS prior_rev
    FROM monthly_sales
)
SELECT
    brand,
    month,
    revenue,
    prior_rev,
    CASE WHEN revenue < prior_rev THEN 'Decline' ELSE 'Growth' END AS trend
FROM changes
WHERE prior_rev IS NOT NULL;
```

---

### Running Totals & Cumulative Aggregates

```sql
-- Running YTD revenue (resets each year)
SELECT
    brand,
    sale_date,
    daily_revenue,
    SUM(daily_revenue) OVER (
        PARTITION BY brand, YEAR(sale_date)
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS ytd_revenue

-- 7-day rolling average (useful for smoothing daily sales noise)
    ,AVG(daily_revenue) OVER (
        PARTITION BY brand
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_7d_avg

-- Running total as % of period total (contribution to YTD)
    ,SUM(daily_revenue) OVER (
        PARTITION BY brand, YEAR(sale_date)
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) / NULLIF(SUM(daily_revenue) OVER (PARTITION BY brand, YEAR(sale_date)), 0) AS pct_of_annual

FROM daily_sales;
```

**ROWS vs RANGE (common interview question):**
- `ROWS` is physical — counts literal rows before/after
- `RANGE` is logical — includes all rows with the same ORDER BY value (can cause unexpected behavior with ties)
- **Always use ROWS unless you explicitly need RANGE behavior**

---

### FIRST_VALUE / LAST_VALUE

```sql
-- First and last sale date per customer
SELECT
    customer_id,
    order_date,
    revenue,
    FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS first_order_date,
    LAST_VALUE(order_date)  OVER (
        PARTITION BY customer_id ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_order_date
FROM orders;
-- Note: LAST_VALUE requires the full frame clause — without it, the default frame ends at the current row
```

---

### NTILE / PERCENT_RANK / CUME_DIST

```sql
-- NTILE: divide into buckets (quartiles, deciles)
SELECT
    store_id,
    revenue,
    NTILE(4)  OVER (ORDER BY revenue DESC) AS revenue_quartile,   -- 1 = top 25%
    NTILE(10) OVER (ORDER BY revenue DESC) AS revenue_decile      -- 1 = top 10%
FROM store_performance;

-- PERCENT_RANK: where does this row fall percentile-wise (0 to 1)
-- CUME_DIST: fraction of rows at or below this value
SELECT
    product_name,
    revenue,
    ROUND(PERCENT_RANK() OVER (ORDER BY revenue), 3) AS percentile_rank,
    ROUND(CUME_DIST()    OVER (ORDER BY revenue), 3) AS cumulative_dist
FROM products;
```

---

### QUALIFY (Snowflake-specific — Key Differentiator)

`QUALIFY` filters on window function results directly — no CTE or subquery needed. Interviewers love this one.

```sql
-- Top 1 product per brand (instead of ROW_NUMBER in a CTE)
SELECT brand, product_name, SUM(revenue) AS total_rev
FROM sales
GROUP BY brand, product_name
QUALIFY ROW_NUMBER() OVER (PARTITION BY brand ORDER BY SUM(revenue) DESC) = 1;

-- Keep only rows where revenue exceeds the brand average
SELECT brand, product_name, revenue
FROM sales
QUALIFY revenue > AVG(revenue) OVER (PARTITION BY brand);
```

**T-Life talking point:** *"I used QUALIFY heavily in Snowflake to clean up what would have been nested CTEs — it's one of Snowflake's best ergonomic features for analytical queries."*

---

## Part 2 — CTEs, Subqueries & Query Structure

### CTE Best Practices

```sql
-- Layered CTEs: each builds on the last — readable, debuggable
WITH raw_sales AS (
    SELECT
        brand,
        channel,
        DATE_TRUNC('month', sale_date) AS sale_month,
        SUM(revenue)                   AS monthly_rev,
        SUM(units_sold)                AS monthly_units
    FROM sales
    WHERE sale_date >= '2025-01-01'
    GROUP BY 1, 2, 3
),
with_prior AS (
    SELECT
        *,
        LAG(monthly_rev) OVER (PARTITION BY brand, channel ORDER BY sale_month) AS prior_month_rev
    FROM raw_sales
),
with_changes AS (
    SELECT
        *,
        ROUND((monthly_rev - prior_month_rev) / NULLIF(prior_month_rev, 0) * 100, 1) AS mom_growth_pct
    FROM with_prior
)
SELECT * FROM with_changes
WHERE mom_growth_pct < -10
ORDER BY mom_growth_pct;
```

**Interview rule:** Lead every complex query with CTEs. Name them descriptively. Tell the interviewer: *"I'll break this into steps so the logic is clear — first I'll build X, then layer Y on top of it."*

---

### Recursive CTEs (know the concept)

Used for hierarchical data — org charts, product category trees, bill of materials.

```sql
-- Walk a product category hierarchy
WITH RECURSIVE category_tree AS (
    -- Anchor: top-level categories
    SELECT category_id, category_name, parent_id, 0 AS depth, category_name AS path
    FROM product_categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive: children
    SELECT c.category_id, c.category_name, c.parent_id,
           ct.depth + 1,
           ct.path || ' > ' || c.category_name
    FROM product_categories c
    JOIN category_tree ct ON c.parent_id = ct.category_id
)
SELECT * FROM category_tree ORDER BY path;
```

**When to mention:** If asked about data modeling or hierarchy rollups in the JD (Planning/Merchandise product hierarchy is likely relevant at Crocs).

---

## Part 3 — JOINs Deep Dive

Know the mechanics AND the business scenarios for each.

```sql
-- INNER JOIN: only matching rows
SELECT o.order_id, p.product_name, o.revenue
FROM orders o
INNER JOIN products p ON o.product_id = p.product_id;

-- LEFT JOIN: all orders, even those without a matching product (data quality check)
SELECT o.order_id, p.product_name, o.revenue
FROM orders o
LEFT JOIN products p ON o.product_id = p.product_id
WHERE p.product_id IS NULL;  -- orphaned orders — product was deleted/not loaded

-- SELF JOIN: compare a row to other rows in the same table
-- Example: find customers who placed orders within 7 days of a prior order
SELECT a.customer_id, a.order_date AS order1, b.order_date AS order2,
       DATEDIFF('day', a.order_date, b.order_date) AS days_apart
FROM orders a
JOIN orders b ON a.customer_id = b.customer_id
             AND b.order_date > a.order_date
             AND DATEDIFF('day', a.order_date, b.order_date) <= 7;

-- ANTI JOIN pattern (customers who never purchased HEYDUDE)
SELECT DISTINCT c.customer_id
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.brand = 'HEYDUDE'
);

-- CROSS JOIN: every combination (use carefully — can explode row count)
-- Example: create a complete date × product grid for gap-filling
SELECT d.date, p.product_id
FROM dim_dates d
CROSS JOIN dim_products p
WHERE d.date BETWEEN '2026-01-01' AND '2026-03-31';
```

---

## Part 4 — NULL Handling (Common Gotcha)

```sql
-- NULLIF: returns NULL if two values are equal (safe division denominator)
SELECT revenue / NULLIF(units, 0) AS avg_price;

-- COALESCE: returns first non-null value
SELECT COALESCE(return_date, ship_date, order_date) AS best_available_date;

-- NVL (Snowflake/Oracle): two-argument COALESCE equivalent
SELECT NVL(discount, 0) AS discount_safe;

-- NULL comparisons: NULL = NULL is FALSE — use IS NULL / IS NOT NULL
WHERE return_reason IS NULL    -- correct
WHERE return_reason = NULL     -- always returns nothing

-- Aggregate behavior: all aggregate functions ignore NULLs except COUNT(*)
SELECT
    COUNT(*)           AS total_rows,     -- counts everything
    COUNT(return_date) AS rows_with_return -- counts non-NULLs only
FROM orders;
```

---

## Part 5 — Retail-Specific Query Patterns

### Sell-Through Rate

```sql
SELECT
    i.product_id,
    p.product_name,
    p.brand,
    i.units_available,
    COALESCE(s.units_sold, 0)                                              AS units_sold,
    ROUND(COALESCE(s.units_sold, 0)::FLOAT
          / NULLIF(i.units_available, 0) * 100, 1)                        AS sell_through_pct,
    DATEDIFF('day', i.receipt_date, CURRENT_DATE())                        AS days_in_inventory
FROM inventory i
JOIN products p ON i.product_id = p.product_id
LEFT JOIN (
    SELECT product_id, SUM(units) AS units_sold
    FROM sales
    WHERE sale_date >= i.receipt_date
    GROUP BY product_id
) s ON i.product_id = s.product_id
ORDER BY sell_through_pct ASC;
```

---

### Year-Over-Year Revenue by Brand & Channel

```sql
WITH annual AS (
    SELECT
        brand,
        channel,
        YEAR(sale_date)    AS sale_year,
        SUM(revenue)       AS annual_rev
    FROM sales
    GROUP BY 1, 2, 3
)
SELECT
    cy.brand,
    cy.channel,
    cy.annual_rev                                                       AS cy_revenue,
    py.annual_rev                                                       AS py_revenue,
    cy.annual_rev - py.annual_rev                                       AS yoy_change,
    ROUND((cy.annual_rev - py.annual_rev) / NULLIF(py.annual_rev, 0) * 100, 1) AS yoy_pct
FROM annual cy
LEFT JOIN annual py
       ON cy.brand = py.brand
      AND cy.channel = py.channel
      AND cy.sale_year = py.sale_year + 1
WHERE cy.sale_year = 2026
ORDER BY brand, yoy_pct DESC;
```

---

### Top N Products per Brand (Multiple Approaches)

```sql
-- Approach 1: ROW_NUMBER in CTE
WITH ranked AS (
    SELECT
        brand,
        product_name,
        SUM(revenue) AS total_rev,
        ROW_NUMBER() OVER (PARTITION BY brand ORDER BY SUM(revenue) DESC) AS rn
    FROM sales
    GROUP BY brand, product_name
)
SELECT brand, product_name, total_rev
FROM ranked
WHERE rn <= 5;

-- Approach 2: QUALIFY (cleaner in Snowflake)
SELECT brand, product_name, SUM(revenue) AS total_rev
FROM sales
GROUP BY brand, product_name
QUALIFY ROW_NUMBER() OVER (PARTITION BY brand ORDER BY SUM(revenue) DESC) <= 5;
```

---

### Margin & Profitability Analysis

```sql
-- Product-level margin with brand/channel context
SELECT
    p.brand,
    p.category,
    p.product_name,
    s.channel,
    SUM(s.revenue)                                                  AS gross_revenue,
    SUM(s.units)                                                    AS units_sold,
    SUM(s.revenue) / NULLIF(SUM(s.units), 0)                       AS avg_selling_price,
    p.standard_cost,
    SUM(s.revenue) - (SUM(s.units) * p.standard_cost)              AS gross_profit,
    ROUND(
        (SUM(s.revenue) - (SUM(s.units) * p.standard_cost))
        / NULLIF(SUM(s.revenue), 0) * 100
    , 1)                                                            AS gross_margin_pct
FROM sales s
JOIN products p ON s.product_id = p.product_id
GROUP BY p.brand, p.category, p.product_name, s.channel, p.standard_cost
ORDER BY gross_margin_pct DESC;
```

---

### Inventory Aging & Slow Movers

```sql
-- Identify dead stock: low sell-through AND old receipt date
WITH sell_through AS (
    SELECT
        i.product_id,
        i.sku,
        i.receipt_date,
        i.units_received,
        COALESCE(SUM(s.units), 0)                                           AS units_sold,
        DATEDIFF('day', i.receipt_date, CURRENT_DATE())                     AS days_on_hand,
        COALESCE(SUM(s.units), 0)::FLOAT / NULLIF(i.units_received, 0)     AS sell_thru_rate
    FROM inventory i
    LEFT JOIN sales s ON i.product_id = s.product_id
                      AND s.sale_date >= i.receipt_date
    GROUP BY i.product_id, i.sku, i.receipt_date, i.units_received
)
SELECT
    st.*,
    p.product_name,
    p.brand,
    CASE
        WHEN days_on_hand > 180 AND sell_thru_rate < 0.30 THEN 'Dead Stock'
        WHEN days_on_hand > 90  AND sell_thru_rate < 0.50 THEN 'Slow Mover'
        ELSE 'Healthy'
    END AS inventory_status
FROM sell_through st
JOIN products p ON st.product_id = p.product_id
ORDER BY days_on_hand DESC;
```

---

### Channel Mix & Revenue Contribution

```sql
-- Revenue by channel as % of brand total, with rank
SELECT
    brand,
    channel,
    SUM(revenue)                                                            AS channel_rev,
    SUM(SUM(revenue)) OVER (PARTITION BY brand)                            AS brand_total_rev,
    ROUND(SUM(revenue) / NULLIF(SUM(SUM(revenue)) OVER (PARTITION BY brand), 0) * 100, 1) AS pct_of_brand,
    RANK() OVER (PARTITION BY brand ORDER BY SUM(revenue) DESC)            AS channel_rank
FROM sales
WHERE YEAR(sale_date) = 2026
GROUP BY brand, channel
ORDER BY brand, channel_rank;
```

---

### Cohort Retention Analysis

```sql
-- Customer retention by acquisition cohort
WITH first_purchase AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', MIN(order_date)) AS cohort_month
    FROM orders
    GROUP BY customer_id
),
activity AS (
    SELECT
        o.customer_id,
        fp.cohort_month,
        DATE_TRUNC('month', o.order_date)                      AS activity_month,
        DATEDIFF('month', fp.cohort_month,
                 DATE_TRUNC('month', o.order_date))            AS months_since_first
    FROM orders o
    JOIN first_purchase fp ON o.customer_id = fp.customer_id
),
cohort_sizes AS (
    SELECT cohort_month, COUNT(DISTINCT customer_id) AS cohort_size
    FROM first_purchase
    GROUP BY cohort_month
)
SELECT
    a.cohort_month,
    a.months_since_first,
    COUNT(DISTINCT a.customer_id)                              AS retained_customers,
    cs.cohort_size,
    ROUND(COUNT(DISTINCT a.customer_id)::FLOAT / cs.cohort_size * 100, 1) AS retention_rate
FROM activity a
JOIN cohort_sizes cs ON a.cohort_month = cs.cohort_month
GROUP BY a.cohort_month, a.months_since_first, cs.cohort_size
ORDER BY a.cohort_month, a.months_since_first;
```

---

### Cross-Brand Purchaser Identification

```sql
-- Customers who bought both Crocs and HEYDUDE (same calendar year)
SELECT DISTINCT c.customer_id
FROM orders c
WHERE c.brand = 'Crocs'
  AND YEAR(c.order_date) = 2026
  AND EXISTS (
      SELECT 1 FROM orders h
      WHERE h.customer_id = c.customer_id
        AND h.brand = 'HEYDUDE'
        AND YEAR(h.order_date) = 2026
  );

-- With their revenue from each brand
WITH cross_shoppers AS (
    SELECT DISTINCT a.customer_id
    FROM orders a
    JOIN orders b ON a.customer_id = b.customer_id
                 AND a.brand != b.brand
                 AND YEAR(a.order_date) = YEAR(b.order_date)
    WHERE a.brand IN ('Crocs', 'HEYDUDE')
      AND b.brand IN ('Crocs', 'HEYDUDE')
      AND YEAR(a.order_date) = 2026
)
SELECT
    cs.customer_id,
    SUM(CASE WHEN brand = 'Crocs'   THEN revenue END) AS crocs_rev,
    SUM(CASE WHEN brand = 'HEYDUDE' THEN revenue END) AS heydude_rev,
    SUM(revenue)                                       AS total_rev
FROM cross_shoppers cs
JOIN orders o ON cs.customer_id = o.customer_id
WHERE YEAR(o.order_date) = 2026
GROUP BY cs.customer_id
ORDER BY total_rev DESC;
```

---

### PIVOT / UNPIVOT (Snowflake)

```sql
-- Pivot: channels become columns
SELECT *
FROM (
    SELECT brand, channel, revenue FROM sales WHERE YEAR(sale_date) = 2026
)
PIVOT (SUM(revenue) FOR channel IN ('DTC', 'Wholesale', 'Marketplace'))
AS p(brand, dtc_rev, wholesale_rev, marketplace_rev);

-- UNPIVOT: flatten wide columns back to rows
SELECT brand, channel, revenue
FROM wide_sales
UNPIVOT (revenue FOR channel IN (dtc_rev, wholesale_rev, marketplace_rev));
```

---

## Part 6 — Snowflake Platform Concepts

### Virtual Warehouses

```sql
-- Key settings to know
ALTER WAREHOUSE bi_warehouse SET
    WAREHOUSE_SIZE = 'MEDIUM'
    AUTO_SUSPEND = 120          -- seconds of inactivity before suspending
    AUTO_RESUME = TRUE;         -- resumes automatically on first query

-- Scale out for concurrency (multi-cluster — not bigger queries)
ALTER WAREHOUSE bi_warehouse SET
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 3
    SCALING_POLICY = 'ECONOMY';   -- ECONOMY waits longer before adding cluster

-- Manually suspend/resume
ALTER WAREHOUSE bi_warehouse SUSPEND;
ALTER WAREHOUSE bi_warehouse RESUME;

-- Check warehouse credit usage
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE warehouse_name = 'BI_WAREHOUSE'
ORDER BY start_time DESC
LIMIT 30;
```

**Warehouse sizing rule of thumb:**
- XS/S: dev, ad hoc small queries, light dashboard loads
- M: typical BI workloads, 10–20 concurrent users
- L: heavy transforms, large aggregations, dbt runs
- **Scale out (multi-cluster), not up, for concurrency problems**

---

### Clustering Keys

Micro-partitions are Snowflake's internal storage unit (~16MB each). Clustering ensures that related data lands in the same micro-partitions so Snowflake can prune more of them at query time.

```sql
-- Add a clustering key (most useful on large tables with repeated range filters)
ALTER TABLE sales CLUSTER BY (brand, DATE_TRUNC('month', sale_date));

-- Check if clustering is helping (AVERAGE_DEPTH close to 1 = well-clustered)
SELECT SYSTEM$CLUSTERING_INFORMATION('sales', '(brand, sale_date)');

-- Automatic Clustering: Snowflake maintains it for you (costs credits)
ALTER TABLE sales RESUME RECLUSTER;
```

**When NOT to cluster:** Small tables (< a few hundred GB), tables with no consistent filter pattern, tables that change structure frequently.

---

### Zero-Copy Cloning

Creates an instant snapshot without duplicating storage — only stores changes (copy-on-write).

```sql
-- Clone a table for dev/testing without moving data
CREATE TABLE sales_dev CLONE sales;

-- Clone an entire database (dev environment in seconds)
CREATE DATABASE crocs_dev CLONE crocs_prod;

-- Clone a schema
CREATE SCHEMA analytics_test CLONE analytics;

-- Point-in-time clone (before a bad operation)
CREATE TABLE sales_backup CLONE sales BEFORE (TIMESTAMP => '2026-05-01 08:00:00'::TIMESTAMP);
```

**Interview talking point:** *"At T-Life I would have loved zero-copy cloning — we spent hours managing dev/prod data sync. In Snowflake you can spin up a complete dev environment in seconds with no storage cost until you start making changes."*

---

### Time Travel

```sql
-- Query historical state
SELECT * FROM sales AT (TIMESTAMP => '2026-04-30 09:00:00'::TIMESTAMP);
SELECT * FROM sales AT (OFFSET => -3600);           -- 1 hour ago
SELECT * FROM sales BEFORE (STATEMENT => '<query_id>'); -- before a specific statement ran

-- Restore from a bad DELETE
CREATE TABLE sales_restored AS
SELECT * FROM sales BEFORE (STATEMENT => '<the_bad_delete_query_id>');

-- Default retention: 1 day (Standard), up to 90 days (Enterprise)
ALTER TABLE sales SET DATA_RETENTION_TIME_IN_DAYS = 14;
```

---

### Views & Materialized Views

```sql
-- Standard view: logical abstraction, no storage, executes on every query
CREATE OR REPLACE VIEW vw_daily_brand_revenue AS
SELECT
    brand,
    DATE_TRUNC('day', sale_date) AS sale_day,
    channel,
    SUM(revenue)                 AS daily_rev,
    SUM(units)                   AS daily_units
FROM sales
GROUP BY 1, 2, 3;

-- Materialized view: pre-computed, auto-refreshes when source changes
-- Use for expensive aggregations queried frequently
CREATE MATERIALIZED VIEW mv_monthly_brand_channel AS
SELECT
    brand,
    channel,
    DATE_TRUNC('month', sale_date) AS sale_month,
    SUM(revenue)                   AS monthly_rev,
    COUNT(DISTINCT order_id)       AS order_count
FROM sales
GROUP BY 1, 2, 3;
-- Note: MVs in Snowflake have limitations — no outer joins, no window functions
```

**Decision rule:** View for simplicity/abstraction. Materialized view when the query is expensive and run frequently. Don't over-use MVs — they cost storage and maintenance credits.

---

### Dynamic Tables (Snowflake — newer feature)

A step beyond materialized views: define a query, set a refresh lag, and Snowflake maintains it automatically. Supports joins, window functions, and complex logic that MVs don't.

```sql
CREATE OR REPLACE DYNAMIC TABLE dt_channel_summary
    LAG = '1 hour'
    WAREHOUSE = 'BI_WAREHOUSE'
AS
SELECT
    brand,
    channel,
    DATE_TRUNC('day', sale_date) AS day,
    SUM(revenue)                 AS daily_rev,
    SUM(units)                   AS daily_units
FROM sales
GROUP BY 1, 2, 3;
```

**When to mention:** Shows you're current with the Snowflake roadmap. *"Dynamic Tables are essentially declarative pipelines — you define the target state and Snowflake handles the incremental refresh. They fill the gap where materialized views had limitations."*

---

### Query Performance Troubleshooting

When a query or dashboard is slow, this is the diagnostic sequence:

1. **Query Profile (Snowflake UI)** — click the query in history, open Profile. Look for:
   - `TableScan` with high percentage = not pruning micro-partitions → add a clustering key or tighten WHERE filters
   - `Aggregate` or `Sort` as the heaviest node = computation bottleneck → pre-aggregate or add a materialized view
   - `SpillToLocal/SpillToRemote` = warehouse is undersized for the data volume → scale up warehouse

2. **Partition pruning check** — in the Profile, look at `partitions scanned / total partitions`. If you're scanning > 50% of a large table, you're missing pruning.

3. **Check result cache** — if the same query returns instantly for some users but slowly for others, dynamic date functions (`CURRENT_DATE()`, `GETDATE()`) bypass the cache. Use date parameters or scheduled extracts instead.

4. **Concurrency vs. query size** — if individual queries are fast but the dashboard times out under load, the problem is concurrency → add a second cluster, not a bigger warehouse size.

5. **ACCOUNT_USAGE views** — query patterns across the org:
```sql
-- Most expensive queries in the last 7 days
SELECT
    query_text,
    execution_time / 1000 AS exec_seconds,
    bytes_scanned / 1e9   AS gb_scanned,
    credits_used_cloud_services
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE start_time > DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY execution_time DESC
LIMIT 20;
```

---

### Role-Based Access Control (RBAC)

```sql
-- Standard grant pattern for a BI analyst role
GRANT USAGE ON DATABASE crocs_db TO ROLE bi_analyst;
GRANT USAGE ON SCHEMA crocs_db.analytics TO ROLE bi_analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA crocs_db.analytics TO ROLE bi_analyst;
GRANT SELECT ON FUTURE TABLES IN SCHEMA crocs_db.analytics TO ROLE bi_analyst;  -- critical

-- Create a service account for Power BI refresh
CREATE USER powerbi_svc_account
    PASSWORD = '...'
    DEFAULT_ROLE = powerbi_role
    DEFAULT_WAREHOUSE = 'BI_WAREHOUSE';

GRANT ROLE powerbi_role TO USER powerbi_svc_account;
```

**Common gap in practice:** Forgetting `GRANT ON FUTURE TABLES` — without it, every new table added to the schema requires a manual re-grant.

---

### Streams & Tasks (Know Conceptually)

- **Stream:** CDC (change data capture) object that tracks inserts/updates/deletes on a table since the last time the stream was consumed. Key columns: `METADATA$ACTION`, `METADATA$ISUPDATE`.
- **Task:** Scheduled SQL execution — runs on a cron or at a set interval. Often used to consume a stream and write to a target table.

```sql
-- Stream on the sales table
CREATE STREAM sales_stream ON TABLE sales;

-- Task to process new records every 15 minutes
CREATE TASK process_new_sales
    WAREHOUSE = 'ETL_WH'
    SCHEDULE = '15 MINUTE'
AS
INSERT INTO sales_gold
SELECT * FROM sales_stream WHERE METADATA$ACTION = 'INSERT';
```

**Interview framing:** *"I haven't built Streams and Tasks pipelines from scratch, but I understand the pattern — Streams as CDC trackers and Tasks as lightweight schedulers. In most of my work the engineering team owned that layer; I consumed the output."*

---

## Part 7 — SQL Performance Optimization Cheat Sheet

| Problem | Diagnosis | Fix |
|---|---|---|
| Slow filter scan | High partitions_scanned in Query Profile | Add clustering key on filter columns |
| Query re-runs slowly | No result cache reuse | Remove `CURRENT_DATE()` from query; use parameter |
| Timeout under concurrent load | Warehouse queuing | Add second cluster to multi-cluster warehouse |
| Expensive aggregation | Aggregate node is heaviest | Pre-aggregate with materialized view or dynamic table |
| Memory spill | SpillToLocal/Remote in profile | Scale up warehouse size |
| Large unfiltered table scan | WHERE clause missing/weak | Add date range filter; verify clustering |
| SELECT * in production | Unnecessary columns loaded | Enumerate only needed columns |
| Division by zero errors | Error in output | Wrap denominator in `NULLIF(col, 0)` |
| JOIN producing too many rows | Unexpected fan-out | Check for non-unique join keys; add DISTINCT or GROUP BY |

---

## Part 8 — Practice Problems (Do These Out Loud)

**Problem 1** — Revenue share with window functions  
*"Write a query that returns each brand's revenue as a % of total enterprise revenue for each month of 2026, plus its rank that month."*

```sql
SELECT
    brand,
    DATE_TRUNC('month', sale_date)                                     AS sale_month,
    SUM(revenue)                                                       AS brand_rev,
    SUM(SUM(revenue)) OVER (PARTITION BY DATE_TRUNC('month', sale_date)) AS total_rev,
    ROUND(SUM(revenue) / NULLIF(SUM(SUM(revenue)) OVER (
        PARTITION BY DATE_TRUNC('month', sale_date)), 0) * 100, 1)    AS rev_share_pct,
    RANK() OVER (PARTITION BY DATE_TRUNC('month', sale_date)
                 ORDER BY SUM(revenue) DESC)                           AS monthly_rank
FROM sales
WHERE sale_date BETWEEN '2026-01-01' AND '2026-12-31'
GROUP BY brand, DATE_TRUNC('month', sale_date)
ORDER BY sale_month, monthly_rank;
```

---

**Problem 2** — MoM decline flag  
*"Find every brand/channel combination where month-over-month revenue declined more than 10% in any month of 2026."*

```sql
WITH monthly AS (
    SELECT
        brand,
        channel,
        DATE_TRUNC('month', sale_date) AS sale_month,
        SUM(revenue)                   AS monthly_rev
    FROM sales
    WHERE YEAR(sale_date) = 2026
    GROUP BY 1, 2, 3
),
with_lag AS (
    SELECT *,
        LAG(monthly_rev) OVER (PARTITION BY brand, channel ORDER BY sale_month) AS prior_rev
    FROM monthly
)
SELECT
    brand,
    channel,
    sale_month,
    monthly_rev,
    prior_rev,
    ROUND((monthly_rev - prior_rev) / NULLIF(prior_rev, 0) * 100, 1) AS mom_pct
FROM with_lag
WHERE prior_rev IS NOT NULL
  AND (monthly_rev - prior_rev) / NULLIF(prior_rev, 0) < -0.10
ORDER BY mom_pct;
```

---

**Problem 3** — Slow-moving inventory  
*"Return products where sell-through rate is below 30% and inventory has been on hand more than 90 days."*

```sql
WITH sold AS (
    SELECT product_id, SUM(units) AS units_sold
    FROM sales
    GROUP BY product_id
)
SELECT
    i.product_id,
    p.product_name,
    p.brand,
    i.units_received,
    COALESCE(s.units_sold, 0)                                                    AS units_sold,
    ROUND(COALESCE(s.units_sold, 0)::FLOAT / NULLIF(i.units_received, 0) * 100, 1) AS sell_thru_pct,
    DATEDIFF('day', i.receipt_date, CURRENT_DATE())                              AS days_on_hand
FROM inventory i
JOIN products p    ON i.product_id = p.product_id
LEFT JOIN sold  s  ON i.product_id = s.product_id
WHERE DATEDIFF('day', i.receipt_date, CURRENT_DATE()) > 90
  AND COALESCE(s.units_sold, 0)::FLOAT / NULLIF(i.units_received, 0) < 0.30
ORDER BY days_on_hand DESC;
```

---

**Problem 4** — Cross-brand purchasers  
*"Identify customers who purchased both Crocs and HEYDUDE in the same calendar year."*

```sql
SELECT DISTINCT o1.customer_id
FROM orders o1
WHERE o1.brand = 'Crocs'
  AND EXISTS (
      SELECT 1 FROM orders o2
      WHERE o2.customer_id = o1.customer_id
        AND o2.brand = 'HEYDUDE'
        AND YEAR(o2.order_date) = YEAR(o1.order_date)
  );
```

---

**Problem 5** — Running YTD with reset  
*"Return daily revenue with a running YTD total that resets on January 1 each year, for each brand."*

```sql
SELECT
    brand,
    sale_date,
    SUM(revenue)                                                    AS daily_rev,
    SUM(SUM(revenue)) OVER (
        PARTITION BY brand, YEAR(sale_date)
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )                                                               AS ytd_rev
FROM sales
GROUP BY brand, sale_date
ORDER BY brand, sale_date;
```

---

**Problem 6** — Snowflake slow dashboard  
*"A Power BI dashboard connected to Snowflake is timing out at peak hours. Walk me through diagnosis and resolution."*

> 1. **Check Snowflake Query Profile** for the specific queries Power BI generates — look for unpartitioned full scans or spill events
> 2. **Check concurrency** — if individual queries are fast but timeout under load, it's a warehouse queuing issue, not query efficiency → enable multi-cluster
> 3. **Check for result cache bypasses** — dynamic date functions like `CURRENT_DATE()` or report-level date slicers that change per user prevent caching → restructure with date parameters or scheduled imports
> 4. **Review Power BI connection mode** — if DirectQuery, every visual interaction fires a Snowflake query; switch to Import with scheduled refresh for most dashboards
> 5. **Add a materialized view or dynamic table** for the most expensive aggregation if it can tolerate slight staleness

---

**Problem 7** — Second-highest revenue product per brand (no RANK shortcut)  
*"Without using a ranking function, how would you find the second-highest revenue product for each brand?"*

```sql
-- Correlated subquery approach
SELECT brand, product_name, total_rev
FROM (
    SELECT brand, product_name, SUM(revenue) AS total_rev
    FROM sales GROUP BY brand, product_name
) t
WHERE total_rev < (
    SELECT MAX(total_rev)
    FROM (
        SELECT brand, product_name, SUM(revenue) AS total_rev
        FROM sales GROUP BY brand, product_name
    ) inner_t
    WHERE inner_t.brand = t.brand
)
QUALIFY ROW_NUMBER() OVER (PARTITION BY brand ORDER BY total_rev DESC) = 1;
```

---

**Problem 8** — Month-flag with CASE  
*"Flag any month/brand combination where both revenue AND units declined vs prior month simultaneously."*

```sql
WITH monthly AS (
    SELECT brand,
           DATE_TRUNC('month', sale_date) AS month,
           SUM(revenue)                   AS rev,
           SUM(units)                     AS units
    FROM sales GROUP BY 1, 2
),
with_lag AS (
    SELECT *,
        LAG(rev)   OVER (PARTITION BY brand ORDER BY month) AS prior_rev,
        LAG(units) OVER (PARTITION BY brand ORDER BY month) AS prior_units
    FROM monthly
)
SELECT
    brand, month, rev, units,
    CASE
        WHEN rev < prior_rev AND units < prior_units THEN 'Double Decline'
        WHEN rev < prior_rev                          THEN 'Revenue Decline Only'
        WHEN units < prior_units                      THEN 'Units Decline Only'
        ELSE 'Growth'
    END AS performance_flag
FROM with_lag
WHERE prior_rev IS NOT NULL
ORDER BY brand, month;
```

---

## Part 9 — Your T-Life SQL Story (Talking Points)

Use these when asked "give me an example of a complex query you've written":

**Story 1 — Semantic model foundation:**
*"At T-Life I owned the Snowflake schema that powered 30 Power BI dashboards serving 100+ daily users including VP and C-suite. The most complex work was building the fact and dimension tables that made DAX measures clean — pushing aggregation logic into Snowflake views meant the semantic model stayed lean and the DAX was readable."*

**Story 2 — Performance problem solved:**
*"We had a dashboard that was timing out during monthly business reviews — exactly when it mattered most. I used the Snowflake Query Profile to identify a table scan on a 500M-row fact table with no partition pruning. Added a clustering key on the date column and created a materialized view for the monthly aggregation. Load time went from 30+ seconds to under 3."*

**Story 3 — Cross-functional metric alignment:**
*"Planning and Finance were using different revenue numbers because they were querying different tables with different filter logic. I built a single certified Snowflake view that enforced the agreed definition — net revenue after returns, excluding intercompany — and made it the single source of truth for all downstream Power BI reports."*
