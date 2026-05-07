# Crocs Technical Prep — DAX & Power BI
*Role: Sr. BI Analyst | Round 2 Technical*  
*Expanded 2026-05-07 — Primary talking points guide*

---

## How to Use This Guide

DAX and Power BI architecture are your deepest lane. When you get a DAX question, write VAR/RETURN by default, narrate what the filter context is doing, and always connect your answer to a business outcome — not just "this is the syntax." Reference T-Life where you owned the semantic model end-to-end.

---

## Part 1 — Filter Context vs Row Context (Know This Cold)

The most important conceptual distinction in DAX. If you can explain this clearly, you signal senior-level thinking immediately.

### The Core Distinction

- **Row context:** Exists inside calculated columns and X-iterator functions. DAX evaluates expression row by row. You can reference column values directly: `Sales[Revenue]`.
- **Filter context:** Exists in measures. Established by slicers, visual filters, report filters, and CALCULATE modifiers. A measure always aggregates within whatever filter context is active.
- **Context transition:** When CALCULATE appears inside a row context (calculated column or iterator), it converts the row context into an equivalent filter context. This is the source of most subtle DAX bugs.

```dax
-- Row context example: calculated column evaluated row by row
Gross Margin = Sales[Revenue] - Sales[COGS]

-- Filter context example: measure responds to slicers/visuals
Total Revenue = SUM(Sales[Revenue])

-- Context transition: inside SUMX (row context), CALCULATE converts each row
-- into a filter context, then evaluates the measure
Weighted Discount Rate =
DIVIDE(
    SUMX(
        Sales,
        Sales[Units] * CALCULATE(AVERAGE(Sales[Discount Rate]))
        -- CALCULATE here transitions row context → filter context for that row
    ),
    SUM(Sales[Units])
)
```

### Why It Matters in Practice

```dax
-- WRONG: this calculated column tries to use a measure in row context without CALCULATE
-- It will return the TOTAL revenue for all rows, not a per-row value
Bad Column = [Total Revenue]  -- Returns same value for every row

-- CORRECT: calculated column using CALCULATE for context transition
Revenue This Category =
CALCULATE(
    SUM(Sales[Revenue]),
    Products[Category] = RELATED(Products[Category])
)
-- Better solution: just use a measure; don't put aggregations in calculated columns
```

**Rule of thumb:** If it aggregates, it's a measure. Use calculated columns only for row-level attributes you'll slice or filter on (like a flag, a bucket label, or a value derived from a relationship).

---

## Part 2 — CALCULATE (The Most Important Function)

CALCULATE does two things simultaneously: (1) evaluates an expression, (2) modifies the filter context for that evaluation.

```dax
-- Basic: override one filter dimension
Crocs Revenue =
CALCULATE(
    SUM(Sales[Revenue]),
    Sales[Brand] = "Crocs"
)

-- Multiple filters: AND logic (all must be true)
DTC Crocs Q1 Revenue =
CALCULATE(
    SUM(Sales[Revenue]),
    Sales[Brand] = "Crocs",
    Sales[Channel] = "DTC",
    Dates[Quarter] = "Q1"
)

-- Using a filter table expression
High Value Orders =
CALCULATE(
    SUM(Sales[Revenue]),
    FILTER(Sales, Sales[Revenue] > 500)
    -- FILTER creates a table; CALCULATE filters the context to only those rows
)
```

### KEEPFILTERS — Intersection Instead of Override

By default, CALCULATE *replaces* existing filters on the modified column. KEEPFILTERS *intersects* — it adds your new filter on top of existing ones.

```dax
-- Without KEEPFILTERS: ignores the brand slicer, always returns Crocs revenue
Crocs Revenue (always) =
CALCULATE(SUM(Sales[Revenue]), Sales[Brand] = "Crocs")

-- With KEEPFILTERS: returns Crocs revenue ONLY if Crocs is selected in the slicer
-- If user selects HEYDUDE in slicer, this returns BLANK
Crocs Revenue (respects slicer) =
CALCULATE(
    SUM(Sales[Revenue]),
    KEEPFILTERS(Sales[Brand] = "Crocs")
)
```

### USERELATIONSHIP — Activate Inactive Relationships

When your model has multiple relationships between two tables (role-playing dimensions), only one can be active. USERELATIONSHIP temporarily activates an inactive one inside a measure.

```dax
-- Date table relates to Sales on OrderDate (active) and ShipDate (inactive)
Revenue by Ship Date =
CALCULATE(
    SUM(Sales[Revenue]),
    USERELATIONSHIP(Sales[ShipDate], Dates[Date])
)
```

### CROSSFILTER — Change Relationship Direction

```dax
-- Force a bidirectional filter for a specific measure (without making the model bidirectional globally)
Products Sold This Region =
CALCULATE(
    DISTINCTCOUNT(Sales[ProductID]),
    CROSSFILTER(Sales[RegionID], Regions[RegionID], BOTH)
)
```

---

## Part 3 — Time Intelligence (Know These Cold)

All time intelligence functions require a proper Date table: contiguous dates, no gaps, marked as a date table, with the Date column as the relationship key.

### Core Functions

```dax
-- Prior year same period
Revenue LY =
CALCULATE(
    [Total Revenue],
    SAMEPERIODLASTYEAR(Dates[Date])
)

-- YoY % with safe blank handling
Revenue YoY % =
VAR cy = [Total Revenue]
VAR py = [Revenue LY]
RETURN
    IF(
        ISBLANK(py) || py = 0,
        BLANK(),
        DIVIDE(cy - py, py)
    )

-- YTD (year-to-date)
Revenue YTD =
TOTALYTD([Total Revenue], Dates[Date])

-- MTD
Revenue MTD = TOTALMTD([Total Revenue], Dates[Date])

-- QTD
Revenue QTD = TOTALQTD([Total Revenue], Dates[Date])

-- Prior period MTD (month-to-date last year)
Revenue MTD LY =
CALCULATE(
    TOTALMTD([Total Revenue], Dates[Date]),
    SAMEPERIODLASTYEAR(Dates[Date])
)
```

### DATEADD — Flexible Period Shifting

```dax
-- Prior month
Revenue Prior Month =
CALCULATE(
    [Total Revenue],
    DATEADD(Dates[Date], -1, MONTH)
)

-- Prior quarter
Revenue Prior Quarter =
CALCULATE(
    [Total Revenue],
    DATEADD(Dates[Date], -1, QUARTER)
)

-- 2 years ago
Revenue 2Y Ago =
CALCULATE(
    [Total Revenue],
    DATEADD(Dates[Date], -2, YEAR)
)

-- MoM % change
Revenue MoM % =
VAR cy = [Total Revenue]
VAR prior = CALCULATE([Total Revenue], DATEADD(Dates[Date], -1, MONTH))
RETURN DIVIDE(cy - prior, prior, BLANK())
```

### DATESINPERIOD — Rolling Windows

```dax
-- Rolling 3 months (trailing 90 days from the last date in context)
Revenue 3M Rolling =
CALCULATE(
    [Total Revenue],
    DATESINPERIOD(Dates[Date], LASTDATE(Dates[Date]), -3, MONTH)
)

-- Rolling 12 months
Revenue 12M Rolling =
CALCULATE(
    [Total Revenue],
    DATESINPERIOD(Dates[Date], LASTDATE(Dates[Date]), -12, MONTH)
)

-- Last 30 days
Revenue Last 30 Days =
CALCULATE(
    [Total Revenue],
    DATESINPERIOD(Dates[Date], LASTDATE(Dates[Date]), -30, DAY)
)
```

### PARALLELPERIOD — Whole-Period Comparison

```dax
-- Revenue for the ENTIRE prior year (not the same number of days)
-- Use PARALLELPERIOD for full-period comparisons; DATEADD for same-length comparisons
Revenue Full Prior Year =
CALCULATE(
    [Total Revenue],
    PARALLELPERIOD(Dates[Date], -1, YEAR)
)
```

### Fiscal Year Time Intelligence

If Crocs' fiscal year doesn't start January 1:

```dax
-- YTD for a fiscal year starting July 1
Revenue Fiscal YTD =
TOTALYTD(
    [Total Revenue],
    Dates[Date],
    "06-30"     -- fiscal year end date (last day of the fiscal year)
)
```

### Running Total That Resets Each Year

```dax
Revenue Running Annual =
CALCULATE(
    [Total Revenue],
    FILTER(
        ALL(Dates),
        Dates[Year] = MAX(Dates[Year])
        && Dates[Date] <= MAX(Dates[Date])
    )
)
```

---

## Part 4 — Iterator Functions (X Functions)

Use iterators when you need to evaluate an expression row-by-row before aggregating. Standard aggregate functions (SUM, COUNT) can't do row-level math.

```dax
-- SUMX: row-level math, then sum
Total Gross Margin =
SUMX(
    Sales,
    Sales[Revenue] - Sales[COGS]
)

-- Why not just use calculated columns?
-- Measures are dynamic and respond to filter context; calculated columns are static.

-- AVERAGEX: weighted average (avg selling price weighted by units sold)
Avg Selling Price =
DIVIDE(
    SUMX(Sales, Sales[Revenue]),
    SUM(Sales[Units]),
    BLANK()
)

-- SUMX over a filtered table
DTC Gross Margin =
SUMX(
    FILTER(Sales, Sales[Channel] = "DTC"),
    Sales[Revenue] - Sales[COGS]
)

-- MAXX / MINX
Largest Single Order =
MAXX(Sales, Sales[Revenue])

-- COUNTX: count rows that meet a condition
Orders Above Average =
COUNTX(
    FILTER(Sales, Sales[Revenue] > AVERAGEX(Sales, Sales[Revenue])),
    Sales[OrderID]
)

-- RANKX: rank products by revenue within current filter context
Product Rank by Revenue =
RANKX(
    ALLSELECTED(Products[ProductName]),
    [Total Revenue],
    ,       -- no explicit value parameter — uses [Total Revenue] for the current row
    DESC,
    Dense   -- Dense = no gaps in ranking (like DENSE_RANK in SQL)
)
```

### TOPN — Top N by a Measure

```dax
-- Revenue from top 5 products only (responds to current filter context)
Top 5 Product Revenue =
CALCULATE(
    [Total Revenue],
    TOPN(5, Products, [Total Revenue], DESC)
)

-- Names of the top 3 products (use in a card or tooltip)
Top 3 Products =
CONCATENATEX(
    TOPN(3, VALUES(Products[ProductName]), [Total Revenue], DESC),
    Products[ProductName],
    ", ",
    [Total Revenue],
    DESC
)
```

---

## Part 5 — ALL, ALLEXCEPT, ALLSELECTED

These are the three most important filter-removal functions. Interviewers love the distinction between ALLSELECTED and ALL.

```dax
-- ALL: removes ALL filters — grand total regardless of any slicers or visual filters
Revenue Grand Total =
CALCULATE([Total Revenue], ALL(Sales))

-- Revenue share of grand total (ignores ALL slicers)
Revenue % of Grand Total =
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALL(Sales))
)

-- ALLEXCEPT: removes all filters EXCEPT specified columns
-- Use for "share within a group" while respecting other filters
Revenue % of Brand =
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALLEXCEPT(Sales, Sales[Brand]))
)
-- If a channel slicer is active, this returns revenue as % of brand within that channel

-- ALLSELECTED: removes visual-level filters but respects slicer selections
-- The "% of what the user has selected" pattern
Revenue % of Selection =
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALLSELECTED(Sales))
)
```

**Interview explanation:** *"ALL is for absolute grand totals — it ignores everything. ALLSELECTED respects slicers but not visual filters — it's the right choice when you want 'percentage of the filtered set the user is looking at.' ALLEXCEPT keeps specific filters in place while removing others — useful for within-group shares like brand mix by channel."*

---

## Part 6 — VAR / RETURN (Always Use)

VAR evaluates once and stores the result — no recalculation, cleaner debugging, easier to read. Always use it for measures with more than one distinct calculation.

```dax
-- Readable, efficient YoY % with blank handling
Revenue YoY % =
VAR current_rev   = [Total Revenue]
VAR prior_rev     = [Revenue LY]
VAR change        = current_rev - prior_rev
VAR yoy_rate      = DIVIDE(change, prior_rev, BLANK())
RETURN
    IF(
        ISBLANK(prior_rev) || prior_rev = 0,
        BLANK(),
        yoy_rate
    )

-- Complex measure with intermediate steps
Margin Performance Band =
VAR margin_pct =
    DIVIDE(
        SUMX(Sales, Sales[Revenue] - Sales[COGS]),
        SUM(Sales[Revenue])
    )
VAR prior_margin =
    CALCULATE(
        DIVIDE(SUMX(Sales, Sales[Revenue] - Sales[COGS]), SUM(Sales[Revenue])),
        SAMEPERIODLASTYEAR(Dates[Date])
    )
VAR margin_change = margin_pct - prior_margin
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(prior_margin),           "No Prior Period",
        margin_change >= 0.05,           "Expanding (+5%+)",
        margin_change >= 0,              "Stable",
        margin_change >= -0.05,          "Compressing",
        "Significantly Compressing"
    )
```

---

## Part 7 — SWITCH (Cleaner Than Nested IF)

```dax
-- Simple equality switch
Channel Display Name =
SWITCH(
    Sales[Channel],
    "DTC",  "Direct to Consumer",
    "WHS",  "Wholesale",
    "MKT",  "Marketplace",
    "Other Channels"    -- default
)

-- SWITCH(TRUE()) for range logic — replaces nested IF chains
Revenue Performance Label =
SWITCH(
    TRUE(),
    [Revenue YoY %] >= 0.20,   "Strong Growth",
    [Revenue YoY %] >= 0.05,   "Growth",
    [Revenue YoY %] >= -0.05,  "Flat",
    [Revenue YoY %] >= -0.20,  "Decline",
    "Significant Decline"
)

-- Dynamic measure selector (with a disconnected table)
Selected KPI =
SWITCH(
    SELECTEDVALUE(KPISelector[KPI]),
    "Revenue",      [Total Revenue],
    "Margin",       [Gross Margin %],
    "Units",        SUM(Sales[Units]),
    "Avg Order",    [Avg Selling Price],
    [Total Revenue]  -- default
)
```

---

## Part 8 — SELECTEDVALUE & Dynamic Titles

### SELECTEDVALUE

Returns the single selected value in the filter context, or a default if multiple/none selected.

```dax
-- Dynamic report title based on slicer selection
Report Title =
"Revenue Dashboard — " &
SELECTEDVALUE(Dates[Year], "All Years") & " | " &
SELECTEDVALUE(Sales[Brand], "All Brands")

-- Conditional measure based on parameter selection
Dynamic Metric =
VAR selected = SELECTEDVALUE(MetricTable[Metric], "Revenue")
RETURN
    SWITCH(
        selected,
        "Revenue",   [Total Revenue],
        "Margin",    [Gross Margin %],
        "Units",     SUM(Sales[Units]),
        [Total Revenue]
    )
```

### HASONEVALUE

Returns TRUE if exactly one value is in the filter context — useful for guarding measures that don't make sense at a total level.

```dax
-- Only show % calculation when a single brand is selected
Brand Revenue Share (Guarded) =
IF(
    HASONEVALUE(Sales[Brand]),
    DIVIDE([Total Revenue], CALCULATE([Total Revenue], ALL(Sales[Brand]))),
    BLANK()
)
```

---

## Part 9 — Disconnected Tables & Parameter Patterns

A disconnected table has no relationship to the data model — it's used purely for slicer-driven dynamic behavior.

```dax
-- Step 1: Create a table in the model (or import a small CSV)
-- MetricSelector: MetricID | MetricName
-- 1 | Revenue
-- 2 | Units
-- 3 | Gross Margin %
-- 4 | Avg Order Value

-- Step 2: Measure that reads the slicer selection
Selected Metric Value =
SWITCH(
    SELECTEDVALUE(MetricSelector[MetricName], "Revenue"),
    "Revenue",         [Total Revenue],
    "Units",           SUM(Sales[Units]),
    "Gross Margin %",  [Gross Margin %],
    "Avg Order Value", [Avg Selling Price],
    [Total Revenue]
)

-- Use case: one chart, one slicer, user picks what to visualize
-- No duplicating charts — massively reduces report complexity
```

**T-Life talking point:** *"At T-Life I used disconnected parameter tables to let executives toggle between revenue, margin, and volume in a single chart without duplicating visuals. It kept the report clean and dramatically reduced the visual count — which also improved load time."*

---

## Part 10 — RELATED, RELATEDTABLE & Virtual Relationships

```dax
-- RELATED: pull a value from the "one" side of a relationship (in calculated column)
Product Category = RELATED(Products[Category])
Product Brand    = RELATED(Products[Brand])

-- RELATEDTABLE: return all matching rows from the "many" side
-- Used on the dimension side
Product Total Orders =
COUNTROWS(RELATEDTABLE(Sales))

Product Revenue =
SUMX(RELATEDTABLE(Sales), Sales[Revenue])
```

### TREATAS — Virtual Relationships

Apply a column's filter context to an unrelated column — useful when you can't create a model relationship.

```dax
-- Apply the Dates[Date] filter to a column in a table with no relationship
Budget vs Actual =
VAR actual = [Total Revenue]
VAR budget =
    CALCULATE(
        SUM(Budget[BudgetAmount]),
        TREATAS(VALUES(Dates[MonthYear]), Budget[MonthYear])
    )
RETURN actual - budget
```

---

## Part 11 — Many-to-Many & Role-Playing Dimensions

### Many-to-Many Relationships

Power BI supports M:M natively via a bridge table, but avoid them when possible — they complicate filter propagation.

```
Sales ←→ SalesTerritory (M:M via bridge table)
Sales --→ Territory (preferred: flatten in SQL before importing)
```

**Interview answer:** *"I prefer to resolve many-to-many at the Snowflake layer — a bridge or aggregated view — rather than in the Power BI model. It keeps the semantic model simpler and DAX measures more predictable."*

### Role-Playing Dimensions

One dimension table serving multiple semantic roles (Order Date, Ship Date, Delivery Date all relating to the same Date table).

```dax
-- Only one relationship can be active; use USERELATIONSHIP for the inactive ones

Revenue by Order Date = [Total Revenue]  -- uses active relationship to OrderDate

Revenue by Ship Date =
CALCULATE(
    [Total Revenue],
    USERELATIONSHIP(Sales[ShipDate], Dates[Date])
)

Revenue by Delivery Date =
CALCULATE(
    [Total Revenue],
    USERELATIONSHIP(Sales[DeliveryDate], Dates[Date])
)
```

**Alternative approach:** Create separate date dimension tables (DimOrderDate, DimShipDate) — avoids USERELATIONSHIP at the cost of model size.

---

## Part 12 — Semantic Model Optimization (Sr. Role Core Topic)

This is the primary responsibility of the Sr. BI Analyst role at Crocs. Know this deeply.

### The VertiPaq Engine

Power BI Import mode compresses data using VertiPaq — a columnar, in-memory engine. Optimization means helping VertiPaq compress data better and do less work at query time.

### Optimization Checklist (Priority Order)

**1. Remove unused columns**
Every column in Import mode is loaded into memory, even if never used in a visual or DAX measure. Audit and delete ruthlessly.

**2. Fix data types**
Integer compresses far better than string. A `1`/`0` flag stored as text ("True"/"False") is far worse than Boolean. Date stored as text is catastrophic.

| Type | Compression | Notes |
|---|---|---|
| Integer | Excellent | Use for IDs, quantities, counts |
| Boolean | Excellent | Flags, yes/no |
| Decimal | Good | Revenue, margin |
| Date | Good | Native date type, not text |
| Text | Varies | Compresses by cardinality — low cardinality text (Brand, Channel) is fine |
| High-card text | Poor | Full SKU names, free-text fields, GUIDs |

**3. Replace calculated columns with measures**
Calculated columns are computed at refresh and stored in memory. Measures compute on demand, in-query. Default to measures unless you genuinely need a per-row attribute.

**4. Disable Auto Date/Time**
Power BI creates hidden date hierarchy tables for every date column unless you turn this off. With 10 date columns and millions of rows, this silently balloons the model. Use your own Date table instead.

```
File → Options → Current File → Data Load → uncheck "Auto date/time"
```

**5. Mark your Date table**
Required for time intelligence functions to work correctly.

**6. Reduce cardinality**
High-cardinality columns compress poorly. Ask whether every product description variant, SKU suffix, or free-form note field actually needs to be in the model.

**7. Aggregation tables**
For large fact tables (100M+ rows), create a pre-aggregated summary table (daily/monthly brand-channel revenue) and configure Power BI aggregations to route coarse-grain queries there automatically.

**8. Hide technical columns**
Foreign keys, surrogate keys, and columns used only in DAX relationships should be hidden from report view — they don't need to be compressed away, but they reduce noise and prevent accidental use.

**9. Bidirectional relationships — use sparingly**
Cross-filtering in both directions is expensive and creates ambiguous filter paths. Only enable bidirectionally when truly necessary (many-to-many bridge tables). Most fact-to-dimension relationships should be single-direction.

**10. Measure formatting**
Always set display format on measures. Unformatted numbers confuse consumers and signal unpolished models.

---

### Import vs DirectQuery vs Composite

```
Import:        Data cached in VertiPaq. Fastest queries. Freshness = last refresh.
DirectQuery:   Every visual fires a live query to source. Always fresh. Can be slow.
Composite:     Import dimensions + DirectQuery facts. Flexible but complex.
               Risk: accidentally sending a DirectQuery to a slow table defeats the point.
```

**Rule for Crocs:**
- Operational dashboards needing daily/hourly refresh → Import + scheduled refresh
- Executive summaries with quarterly data → Import, weekly refresh is fine
- Real-time inventory/live transactional → DirectQuery on Snowflake SQL warehouse (set clustering keys first)
- Large fact (100M+ rows) + small frequently-updated dimensions → Composite model with aggregation tables

---

## Part 13 — Row-Level Security (RLS)

```dax
-- Static role: always show only HEYDUDE data (in the Manage Roles editor)
[Brand] = "HEYDUDE"

-- Dynamic RLS: each user sees only their region
-- Requires a UserMapping table: Email | Region
[Region] = LOOKUPVALUE(
    UserMapping[Region],
    UserMapping[Email], USERPRINCIPALNAME()
)

-- Multi-value dynamic (user can see multiple regions)
-- Requires UserMapping: Email | Region (one row per user-region pair)
COUNTROWS(
    FILTER(
        UserMapping,
        UserMapping[Email] = USERPRINCIPALNAME()
        && UserMapping[Region] = Sales[Region]
    )
) > 0
```

**Testing:** Use "View as role" in Power BI Desktop before publishing. Validate that RLS doesn't accidentally return blank for users with valid access.

**Common mistake:** Applying RLS to the fact table instead of the dimension. Filter the dimension → the relationship propagates to the fact automatically.

---

## Part 14 — Performance Diagnostics

### Performance Analyzer (Built-In)

```
View → Performance Analyzer → Start Recording → Refresh Visuals → Stop
```

What to look at:
- **DAX query** high → optimize the measure or model (most common)
- **Visual display** high → too many visuals, too complex rendering (reduce visuals per page)
- **Other** high → Power BI service overhead, not a DAX/model issue

### DAX Studio (External)

External tool that connects to a running Power BI model and lets you:
- Run DAX queries manually and see execution plans
- Capture the actual DAX Power BI sends to VertiPaq when you interact with a visual
- See storage engine vs formula engine query splits
- Identify scan vs. formula engine bottlenecks

**Interview answer if asked about slow reports:**
> 1. Open Performance Analyzer — identify the slowest visual
> 2. Copy DAX query from Performance Analyzer into DAX Studio
> 3. Run with Server Timings tab open — look at Storage Engine vs Formula Engine time
> 4. If SE is slow: model problem (cardinality, data types, aggregations)
> 5. If FE is slow: DAX problem (complex iterators, context transitions, missing VAR)
> 6. Check model size with VertiPaq Analyzer — identify the largest columns by cardinality and size

---

## Part 15 — Power BI Service & Deployment

### Deployment Pipelines

Three-stage dev → test → prod workflow (requires Premium or PPU per user).

```
[Dev Workspace] → Promote → [Test Workspace] → Promote → [Prod Workspace]
```

- Each stage is a separate workspace with its own dataset/reports
- **Dataset rules** allow different data source connections per stage (dev Snowflake vs prod Snowflake)
- Selective deployment — promote only changed datasets, reports, or dashboards
- Supports rollback by re-deploying the prior stage version

**Scripted gap answer:**
> *"I've managed dev/prod promotion manually — publishing to separate workspaces with separate Snowflake connection configs. I've studied the Deployment Pipeline feature and understand the stage-based workflow, dataset rules, and selective deployment. It's the formalized version of what I was doing manually, and it's one of my top priorities to get hands-on with in a sandbox environment."*

### Tabular Editor (External Tool)

Connects to the semantic model via XMLA endpoint. Three core uses:

1. **Best Practice Analyzer** — runs governance rules (missing measure descriptions, wrong data types, unused columns, measures without format strings)
2. **Scripting with C#** — bulk rename measures, batch-apply formatting, create calculation group items programmatically
3. **Calculation groups** — dynamic format strings and time intelligence switching without duplicating measures

**Scripted gap answer:**
> *"Tabular Editor wasn't in the toolchain at T-Mobile, but I know what it solves: BPA for model governance and scripting for bulk model changes that would be tedious through the UI. I'd treat it as a force multiplier on semantic model work I already know how to do — it automates the maintenance overhead."*

### Shared Semantic Models (Reusable Datasets)

The correct enterprise architecture for large BI teams:

```
One certified semantic model (owned by Sr. BI) → Published to workspace
↓
Multiple report authors build reports by connecting to the shared model
(not building their own — eliminates metric divergence)
```

**Interview talking point:** *"At T-Life I was the sole semantic model owner — every report in the team pointed at one model. That single-source architecture is what I'd want to establish at Crocs, especially across Planning and Merchandise where metric definitions need to be locked."*

---

## Part 16 — Report Design & Governance Principles

### Design Principles (May Come Up)

- **One question per page** — don't put everything on one canvas; use drill-through for detail
- **Drill-through over drill-down** when dimensions differ (Product → Order detail is drill-through, not drill-down)
- **Bookmarks** for toggling views — change visual visibility without refreshing data
- **Tooltip pages** for contextual detail that keeps the main canvas clean
- **Card visuals are expensive** — each fires a separate DAX query; a table or matrix with the same metrics is one query
- **Slicers add filter overhead** — a page with 8 slicers runs 8+ additional queries on load; consolidate where possible
- **Conditional formatting > calculated columns for color** — use measure-based rules, not a hardcoded column

### Governance & Documentation

- Every published measure should have a description (visible in field list tooltip)
- Naming conventions: measures in brackets `[Total Revenue]`, tables in PascalCase `FactSales`, columns in TitleCase `OrderDate`
- Hide all FK/surrogate key columns from report view
- Certify the shared dataset in the Power BI service — signals to report authors it's the official source
- Document metric definitions in a Data Dictionary (separate from the model — living document)

---

## Part 17 — Practice Problems (Do These Out Loud)

**Problem 1** — YoY with blank protection  
*"Revenue for current period vs same period last year, returning BLANK if no prior year data exists."*

```dax
Revenue YoY % =
VAR cy      = [Total Revenue]
VAR py      = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Dates[Date]))
RETURN
    IF(ISBLANK(py) || py = 0, BLANK(), DIVIDE(cy - py, py))
```

---

**Problem 2** — Brand revenue share respecting slicers but not brand filter  
*"Each brand's % of total revenue for whatever the user has selected in other slicers."*

```dax
Revenue % of Selection =
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALLSELECTED(Sales[Brand]))
)
```

---

**Problem 3** — Running DTC revenue resetting each year  
*"Running total of DTC revenue by month that resets at the start of each year."*

```dax
DTC Revenue YTD =
CALCULATE(
    TOTALYTD(
        CALCULATE([Total Revenue], Sales[Channel] = "DTC"),
        Dates[Date]
    )
)

-- If fiscal year end isn't Dec 31:
DTC Revenue Fiscal YTD =
CALCULATE(
    TOTALYTD(
        CALCULATE([Total Revenue], Sales[Channel] = "DTC"),
        Dates[Date],
        "06-30"
    )
)
```

---

**Problem 4** — Top 5 products by revenue in current filter context  
*"Revenue from only the top 5 products, responding to whatever filters are active."*

```dax
Top 5 Product Revenue =
CALCULATE(
    [Total Revenue],
    TOPN(5, ALL(Products[ProductName]), [Total Revenue], DESC)
)
```

---

**Problem 5** — Gross margin % with iterator  
*"Gross margin percentage, calculated correctly at row level before aggregating."*

```dax
Gross Margin % =
VAR gross_margin = SUMX(Sales, Sales[Revenue] - Sales[COGS])
VAR total_rev    = SUM(Sales[Revenue])
RETURN
    DIVIDE(gross_margin, total_rev, BLANK())
```

---

**Problem 6** — Dynamic metric selector  
*"One measure that returns Revenue, Units, or Avg Order Value based on a slicer selection."*

```dax
Selected KPI =
SWITCH(
    SELECTEDVALUE(MetricSelector[Metric], "Revenue"),
    "Revenue",        [Total Revenue],
    "Units",          SUM(Sales[Units]),
    "Avg Order Value", DIVIDE([Total Revenue], DISTINCTCOUNT(Sales[OrderID])),
    [Total Revenue]
)
```

---

**Problem 7** — Slow page diagnosis  
*"Your report has 15 card visuals and loads in 8 seconds. Walk me through diagnosis and fix."*

> 1. Open Performance Analyzer — which card(s) have the highest DAX query time?
> 2. Copy the slowest card's DAX query into DAX Studio — check if formula engine is doing heavy lifting (complex context transitions, iterators)
> 3. Simplify the underlying measure — break into VARs, check for redundant CALCULATE calls
> 4. Consolidate — replace 15 individual cards with a single matrix/table; one query instead of 15
> 5. Check if cards reference measures that scan a full unfiltered fact table — add aggregation logic or pre-aggregated source tables
> 6. Use VertiPaq Analyzer to check if fact table columns used in those measures have high cardinality text that's compressing poorly

---

**Problem 8** — ALLSELECTED vs ALL  
*"What's the difference between ALLSELECTED and ALL? When do you use each?"*

> ALL removes every filter from the model context — slicers, visual filters, report filters, all of it. Use it when you need an absolute grand total that's anchor-stable regardless of what the user selects.

> ALLSELECTED removes visual-level filters (like a chart's own category axis) but preserves slicer and report-level filters. Use it for "percentage of what the user has currently filtered to" — the denominator follows the user's slicer selections without being affected by the chart itself.

> Example: in a bar chart showing revenue by brand, ALLSELECTED in the denominator returns each brand as % of the slicer-filtered total. ALL in the denominator returns % of the absolute grand total including all brands and dates.

---

**Problem 9** — Context transition gotcha  
*"Why does this calculated column return the same value for every row?"*

```dax
-- This is wrong:
Revenue Column = [Total Revenue]
-- [Total Revenue] is a measure. In a calculated column (row context),
-- the measure evaluates with NO filter context on Sales — it sees the entire table.
-- Every row gets the grand total.

-- If you need per-row context, be explicit:
Revenue Column =
CALCULATE(
    SUM(Sales[Revenue]),
    FILTER(Sales, Sales[OrderID] = EARLIER(Sales[OrderID]))
)
-- Or better yet — this should just be a measure, not a calculated column.
```

---

**Problem 10** — RLS setup  
*"Walk me through setting up dynamic RLS for a regional manager who should only see their region."*

> 1. Create a UserMapping table: `Email` | `Region` — one row per user-region pair
> 2. In Power BI Desktop → Modeling → Manage Roles → New Role: `RegionalManager`
> 3. Apply filter on the Geography dimension (not the fact table):
>    ```dax
>    [Region] = LOOKUPVALUE(UserMapping[Region], UserMapping[Email], USERPRINCIPALNAME())
>    ```
> 4. The model relationship from Geography to Facts propagates the filter — don't apply RLS to the fact directly
> 5. Test with "View as role" in Desktop using test email addresses
> 6. Publish → workspace → Dataset Settings → assign users to the `RegionalManager` role

---

## Part 18 — Your T-Life DAX Story (Talking Points)

**Story 1 — Sole semantic model owner:**
*"At T-Life I was the sole owner of the Power BI semantic model that powered 30 dashboards and received 100+ daily visits from VP and C-suite. That meant I was accountable for every metric definition — if a number was wrong, it came back to me. I built the semantic model with a star schema on top of Snowflake, and treated the DAX measure layer as the single source of truth for all KPI definitions."*

**Story 2 — Time intelligence depth:**
*"The most used features in our executive dashboards were YoY comparisons and rolling 12-month trends. I built a full time intelligence suite — YoY %, MTD, QTD, YTD, and rolling windows — all with blank protection and fiscal period variants. When the Finance team needed a fiscal year that didn't match calendar year, I extended the Date table and adjusted the TOTALYTD calls rather than duplicating measures."*

**Story 3 — Performance optimization:**
*"We had a semantic model that had grown organically and was starting to hit refresh time limits. I did a full VertiPaq audit with DAX Studio — found 40+ unused columns that had been imported but never used in a visual or DAX measure, and two high-cardinality text columns that were compressing poorly. After cleanup, model size dropped 35% and refresh time cut in half."*

**Story 4 — Metric alignment (governance):**
*"Different teams were using different revenue figures in their own reports. I worked with Finance to document the authoritative revenue definition — net of returns, excluding intercompany transfers — and encoded it as a single CALCULATE-based measure in the shared semantic model. Once I published and certified the dataset, every downstream report inherited the same logic. The disagreements in exec meetings stopped."*
