# Crocs Technical Prep — Python & Databricks
**Role:** Sr. Business Intelligence Analyst | Broomfield, CO  
**Stack:** Power BI + Snowflake + **Databricks (preferred)**  
**Date Drafted:** 2026-05-07

---

## Context for This Guide

Databricks is listed as **preferred, not required** for this role. The interview is unlikely to go deep on distributed systems engineering — focus on how Databricks fits a BI/analytics workflow, how you'd connect it to Power BI, and how it compares to Snowflake. Python will come up in the context of notebooks, data wrangling, and maybe lightweight pipeline work — not software engineering.

**Your honest position:** You have exposure to Databricks in the context of connecting to Power BI and understand the Lakehouse architecture conceptually. Your Python strength is in analytics/data wrangling (pandas, notebooks), not software engineering.

---

## Part 1 — Databricks Core Concepts

### 1.1 What Is Databricks?

Databricks is a **unified data analytics platform** built on Apache Spark, designed around the concept of a **Lakehouse** — combining the scalability of a data lake with the structure and governance of a data warehouse.

Key differentiators vs. Snowflake:

| | Databricks | Snowflake |
|---|---|---|
| Primary strength | ML/AI, batch pipelines, data engineering | SQL analytics, warehousing, BI |
| Storage format | **Delta Lake** (open format, Parquet + transaction log) | Proprietary storage |
| Compute model | Spark clusters (auto-scaling) | Virtual warehouses |
| SQL support | Databricks SQL (strong, but secondary) | SQL-first |
| Python/ML | Native (PySpark, MLflow) | Limited (Snowpark) |
| Power BI integration | DirectQuery via JDBC/ODBC or Partner Connect | Native connector |

**Interview framing:** *"I've worked in shops where Snowflake owned the BI layer and Databricks handled the upstream pipeline work — they complement each other. Databricks is where the data scientists and engineers clean and model the data; Snowflake is where I query it for Power BI. That said, Databricks SQL warehouses are mature enough to serve Power BI directly for use cases that don't need a Snowflake hop."*

---

### 1.2 Medallion Architecture (Bronze / Silver / Gold)

The standard pattern for organizing data in a Databricks Lakehouse:

```
Raw sources → [Bronze] → [Silver] → [Gold] → BI/ML
```

| Layer | What it holds | Transformation |
|---|---|---|
| **Bronze** | Raw ingested data, as-is | None — append-only |
| **Silver** | Cleaned, deduplicated, typed | Filtering, standardization, joins |
| **Gold** | Aggregated, business-ready | Dimensional models, KPIs, report-ready sets |

**Why this matters for BI:** Gold layer tables are what you'd point Power BI at — either via Databricks SQL or after syncing to Snowflake. Understanding this helps you speak to data freshness, lineage, and why a metric looks a certain way.

**Interview answer:** *"At T-Life, we effectively had a similar layered approach — raw tables in Snowflake, transformation views in the silver equivalent, and semantic model tables in the gold equivalent. In a Databricks shop I'd own the gold layer definition and make sure Power BI's semantic model sits on top of well-governed, curated tables — not raw bronze."*

---

### 1.3 Delta Lake

Delta Lake is the open-source storage layer that makes Databricks reliable for analytics:

- **ACID transactions** — multiple writers don't corrupt the table
- **Time travel** — query a table as it existed at a prior version: `SELECT * FROM sales VERSION AS OF 5`
- **Schema enforcement** — rejects writes that violate the table schema (like a DB constraint)
- **Schema evolution** — can add new columns without breaking existing readers
- **OPTIMIZE / ZORDER** — compacts small files and co-locates related data for faster queries (similar concept to Snowflake clustering)

```sql
-- Time travel in Delta
SELECT * FROM crocs.gold.sales TIMESTAMP AS OF '2026-04-01';
SELECT * FROM crocs.gold.sales VERSION AS OF 10;

-- Optimize with Z-ordering (like clustering in Snowflake)
OPTIMIZE crocs.gold.sales ZORDER BY (region, product_id);

-- Vacuum old files (like PURGE in Snowflake)
VACUUM crocs.gold.sales RETAIN 168 HOURS;
```

---

### 1.4 Unity Catalog

Databricks' **governance and metastore layer** — the equivalent of Snowflake's role-based access + database/schema hierarchy.

Structure: `catalog.schema.table` (three-level namespace, same as Snowflake's `database.schema.table`)

Key features:
- Fine-grained access control (row/column-level security)
- Data lineage — automatic tracking of what tables feed what downstream
- Audit logs
- Cross-workspace sharing

**Interview framing:** *"Unity Catalog matters to me as a BI lead because it's where I'd define who can see what — especially for sensitive retail data like margins or planning targets. It's analogous to Snowflake's RBAC but with richer lineage."*

---

### 1.5 Databricks SQL & Power BI Integration

Two main ways to connect Power BI to Databricks:

**Option A — Databricks SQL Warehouse (recommended for BI)**
- Create a serverless or pro SQL warehouse in Databricks
- Use the native **Databricks connector in Power BI Desktop**
- Supports both Import and DirectQuery modes
- Works with Unity Catalog tables

**Option B — Databricks Partner Connect**
- One-click integration from Databricks UI
- Sets up the connector and credentials automatically

**DirectQuery considerations (same as Snowflake):**
- Real-time data but slower dashboards
- Push query optimization to the source (Delta table layout matters)
- Use aggregation tables + composite models to reduce query load

```
Power BI Desktop → Get Data → Databricks
→ Server: adb-<workspace-id>.azuredatabricks.net
→ HTTP Path: /sql/1.0/warehouses/<warehouse-id>
→ Auth: Personal Access Token or Azure AD
```

**Interview tip:** If asked about Databricks + Power BI, lead with the SQL Warehouse path — it's the BI-optimized approach, not running queries against raw Spark clusters.

---

### 1.6 Databricks vs. Snowflake — When to Use Which

| Scenario | Use Databricks | Use Snowflake |
|---|---|---|
| ML model training | ✅ | ❌ |
| Streaming ingestion | ✅ (Structured Streaming) | Limited |
| Complex SQL analytics | ✅ (SQL Warehouse) | ✅ (native) |
| Power BI semantic layer | Both work | Preferred for mature BI shops |
| Data science notebooks | ✅ | Limited (Snowpark) |
| Governed data sharing | Unity Catalog | Snowflake Secure Share |
| ELT pipeline orchestration | ✅ (DLT, Jobs) | ✅ (Tasks, dbt) |

**Your honest answer if asked preference:** *"I'm more native in Snowflake for the BI layer, but I understand Databricks' SQL Warehouse is production-ready for Power BI. If the source of truth lives in Databricks gold tables, I'd connect there directly rather than adding a Snowflake hop unnecessarily."*

---

## Part 2 — Python for BI Analytics

### 2.1 Your Python Position

For a Sr. BI Analyst role, Python is expected to be a **scripting and analysis tool**, not production software engineering. You're comfortable with:

- **pandas** for data wrangling and ad hoc analysis
- **Python notebooks** (Jupyter / Databricks) for exploratory work
- **PySpark basics** in a Databricks context
- **Automation scripts** — pulling data, formatting reports, moving files

You are NOT expected to be a software engineer. Frame Python as: *"I use Python for tasks that would be clunky in SQL — reshaping data, automating recurring extracts, prototype calculations before I formalize them in DAX."*

---

### 2.2 pandas — Core Operations

```python
import pandas as pd

# Load data
df = pd.read_csv("sales.csv")
df = pd.read_excel("sales.xlsx", sheet_name="Q1")

# Inspect
df.head(10)
df.dtypes
df.describe()
df.isnull().sum()

# Filter rows
df_q1 = df[df["quarter"] == "Q1"]
df_large = df[(df["revenue"] > 100000) & (df["region"] == "AMER")]

# Select columns
df[["product", "revenue", "units"]]

# Rename columns
df.rename(columns={"old_name": "new_name"}, inplace=True)

# New calculated column
df["margin_pct"] = (df["revenue"] - df["cost"]) / df["revenue"]

# GroupBy aggregation
summary = df.groupby(["region", "category"]).agg(
    total_rev=("revenue", "sum"),
    avg_units=("units", "mean"),
    order_count=("order_id", "count")
).reset_index()

# Pivot table
pivot = df.pivot_table(
    values="revenue",
    index="region",
    columns="quarter",
    aggfunc="sum",
    fill_value=0
)

# Merge (JOIN equivalent)
merged = pd.merge(df_orders, df_products, on="product_id", how="left")

# Sort
df.sort_values("revenue", ascending=False).head(10)

# Export
df.to_csv("output.csv", index=False)
df.to_excel("output.xlsx", index=False)
```

**Likely interview question:** *"Walk me through how you'd use Python to prepare a dataset for analysis."*
→ Answer: Load with pandas, inspect shape/nulls/dtypes, clean (fillna, type cast), aggregate with groupby, export or hand off to Power BI via a parameterized refresh.

---

### 2.3 PySpark in Databricks — Key Differences from pandas

PySpark uses a distributed DataFrame that looks similar to pandas but has important differences:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum, avg, count, when, lit

# In Databricks — SparkSession is pre-initialized as `spark`

# Read a Delta table
df = spark.table("crocs.gold.sales")

# Or read from a path
df = spark.read.format("delta").load("/mnt/gold/sales")

# Inspect
df.printSchema()
df.show(10)
df.count()

# Filter
df_q1 = df.filter(col("quarter") == "Q1")
df_large = df.filter((col("revenue") > 100000) & (col("region") == "AMER"))

# Select
df.select("product", "revenue", "units")

# New column
df = df.withColumn("margin_pct", (col("revenue") - col("cost")) / col("revenue"))

# GroupBy aggregation
summary = df.groupBy("region", "category").agg(
    sum("revenue").alias("total_rev"),
    avg("units").alias("avg_units"),
    count("order_id").alias("order_count")
)

# Join
merged = df_orders.join(df_products, on="product_id", how="left")

# Write back to Delta
summary.write.format("delta").mode("overwrite").saveAsTable("crocs.gold.region_summary")
```

**Key differences pandas vs. PySpark:**

| | pandas | PySpark |
|---|---|---|
| Execution | Eager (runs immediately) | Lazy (builds query plan, runs on action) |
| Size limit | Fits in memory | Distributed — handles TB+ |
| Syntax | `df["col"]` / `df.col` | `col("col")` from pyspark.sql.functions |
| Index | Row index built-in | No row index |
| Speed (small data) | Faster | Overhead from Spark |

**Interview framing:** *"In a Databricks notebook I'd use PySpark for anything hitting large Delta tables, and convert to pandas with `.toPandas()` only for final small result sets I'm exporting or visualizing."*

---

### 2.4 SQL in Databricks Notebooks

You can mix SQL and Python in the same notebook:

```python
# Python cell
df = spark.table("crocs.gold.sales")

# SQL cell (magic command)
%sql
SELECT region, SUM(revenue) AS total_rev
FROM crocs.gold.sales
WHERE quarter = 'Q1'
GROUP BY region
ORDER BY total_rev DESC
```

Or run SQL from Python:
```python
result = spark.sql("""
    SELECT region, SUM(revenue) AS total_rev
    FROM crocs.gold.sales
    WHERE quarter = 'Q1'
    GROUP BY region
    ORDER BY total_rev DESC
""")
result.show()
```

---

### 2.5 Python Automation Patterns (BI-relevant)

**Parameterized data pull:**
```python
import pandas as pd
import snowflake.connector

def pull_sales(start_date: str, end_date: str) -> pd.DataFrame:
    conn = snowflake.connector.connect(
        user="jp.hebert", account="crocs.us-east-1",
        warehouse="BI_WH", database="ANALYTICS", schema="GOLD"
    )
    query = f"""
        SELECT order_date, region, product, revenue
        FROM sales
        WHERE order_date BETWEEN '{start_date}' AND '{end_date}'
    """
    df = pd.read_sql(query, conn)
    conn.close()
    return df
```

**Conditional column (equivalent to IF/SWITCH in DAX):**
```python
import numpy as np

df["tier"] = np.where(df["revenue"] > 500000, "Enterprise",
             np.where(df["revenue"] > 100000, "Mid-Market", "SMB"))

# Or with pd.cut for binning
df["revenue_band"] = pd.cut(
    df["revenue"],
    bins=[0, 50000, 200000, 500000, float("inf")],
    labels=["<50K", "50-200K", "200-500K", "500K+"]
)
```

**Date manipulation:**
```python
df["order_date"] = pd.to_datetime(df["order_date"])
df["year"] = df["order_date"].dt.year
df["month"] = df["order_date"].dt.month
df["quarter"] = df["order_date"].dt.quarter
df["week"] = df["order_date"].dt.isocalendar().week

# Days since last order (recency)
df["days_since_order"] = (pd.Timestamp.today() - df["order_date"]).dt.days
```

---

## Part 3 — Practice Questions

### Databricks / Architecture

**Q1: How would you connect Power BI to Databricks?**
> Use the Databricks SQL Warehouse — it's the BI-optimized compute layer. In Power BI Desktop, use the native Databricks connector with the workspace server URL and HTTP path for the warehouse. For production reports I'd evaluate DirectQuery vs. Import based on data size and refresh frequency. For large fact tables I'd use Import with scheduled refreshes; for smaller, frequently-updated dimensions I might use DirectQuery or a composite model.

**Q2: What is the medallion architecture and how does it affect your work as a BI analyst?**
> Bronze/silver/gold layered pipeline. As a BI analyst I care most about the gold layer — that's where curated, aggregated, governed tables live that I'd build semantic models on top of. Understanding the architecture helps me trace data quality issues upstream and set expectations with stakeholders about data freshness (bronze is near-real-time; gold may be a batch job behind).

**Q3: What's the difference between Delta Lake and a regular Parquet table?**
> Delta adds a transaction log on top of Parquet files. That gives you ACID guarantees, time travel, schema enforcement, and efficient updates/deletes — things raw Parquet can't do. For BI it means I can trust that the table I'm querying wasn't partially written when my report refreshed.

**Q4: When would you choose Databricks over Snowflake for a Power BI source?**
> If the gold layer already lives in Databricks Delta tables and the team has a mature SQL warehouse setup, I'd connect Power BI directly there and avoid the ETL cost of syncing to Snowflake. If the org has heavier BI governance, cost transparency by warehouse, or existing Snowflake expertise, I'd prefer Snowflake. Practically I'd go where the best-governed, most reliable data lives.

---

### Python

**Q5: How do you handle missing values in a dataset?**
```python
df.isnull().sum()              # count nulls per column
df.dropna(subset=["revenue"])  # drop rows where revenue is null
df["cost"].fillna(0)           # fill nulls with 0
df["region"].fillna("Unknown") # fill with a default string
df["revenue"].fillna(df["revenue"].median())  # fill with median
```
> Strategy depends on the column: for metrics I fill with 0 or median; for dimensions I fill with "Unknown" and flag it for the data team to fix upstream.

**Q6: Walk through a groupby aggregation.**
```python
# Revenue by region and quarter, with count of orders
result = df.groupby(["region", "quarter"]).agg(
    total_revenue=("revenue", "sum"),
    order_count=("order_id", "nunique"),
    avg_order_value=("revenue", "mean")
).reset_index()
```

**Q7: How do you join two DataFrames?**
```python
# Left join orders to product dimension
df_final = pd.merge(
    df_orders,
    df_products[["product_id", "category", "brand"]],
    on="product_id",
    how="left"
)
```

**Q8: What does `.reset_index()` do?**
> After a groupby, the grouped columns become the index. `.reset_index()` promotes them back to regular columns, which is almost always what you want for further operations or export.

---

## Part 4 — Gaps to Acknowledge

| Gap | Scripted Answer |
|---|---|
| **Production Databricks pipelines** | "I've worked with Databricks primarily for analytics — connecting to Power BI and querying gold tables. I haven't built Delta Live Table pipelines from scratch, but I understand the architecture and would be comfortable picking it up." |
| **PySpark at scale** | "My PySpark experience is at the analytics layer — querying existing tables, not optimizing distributed jobs. For that level of Spark tuning I'd partner with the data engineering team." |
| **MLflow / Databricks ML** | "I'm familiar with what MLflow does — experiment tracking, model registry — but I haven't used it directly. That's firmly in the data science lane; I'd consume model outputs, not build them." |

---

## Part 5 — Key Talking Points to Weave In

1. **T-Life Databricks context:** *"At T-Life, our Snowflake environment was downstream of a Databricks pipeline — I worked closely with the data engineering team to understand how gold tables were built so I could design the right semantic model on top of them."*

2. **Retail data volumes:** *"In footwear retail, order and transaction data can be substantial — millions of rows per day across a global supply chain. Understanding whether to use DirectQuery or Import from Databricks SQL is a real architectural decision with latency and cost implications."*

3. **Python as a BI tool, not an engineering tool:** *"I use Python when SQL gets unwieldy — reshaping pivot outputs, automating a recurring data pull, or doing exploratory analysis before I formalize something in DAX. It's part of my toolkit, not my primary language."*
