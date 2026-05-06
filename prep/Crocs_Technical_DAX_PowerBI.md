# Crocs Technical Prep — DAX & Power BI
*Role: Sr. BI Analyst | Round 2 Technical*

---

## DAX — Core Concepts

### Filter Context vs Row Context
The most important conceptual distinction in DAX. Get this wrong in an interview and it signals beginner-level.

- **Row context:** Exists inside calculated columns and iterator functions. DAX evaluates one row at a time.
- **Filter context:** Exists in measures. Determined by the visual, slicers, and CALCULATE filters. A measure always evaluates within a filter context.
- **Context transition:** When you use CALCULATE inside a calculated column or iterator, it converts the row context into a filter context. This is the source of most DAX bugs.

```dax
-- Measure: filter context — the SUM responds to whatever filters are active in the report
Total Revenue = SUM(Sales[Revenue])

-- Calculated column: row context — Revenue * Qty evaluated row by row
Gross Sales = Sales[Revenue] * Sales[Quantity]
```

**Rule of thumb:** Use measures for anything that aggregates. Use calculated columns only when you need a value per row that you'll filter or slice on.

---

### CALCULATE — The Most Important DAX Function
CALCULATE modifies the filter context of any expression.

```dax
-- Override a filter
Crocs Brand Revenue =
CALCULATE(
    SUM(Sales[Revenue]),
    Sales[Brand] = "Crocs"
)

-- Remove all filters on a column
Revenue All Channels =
CALCULATE(
    SUM(Sales[Revenue]),
    ALL(Sales[Channel])
)

-- Multiple filters (AND logic)
DTC Crocs Revenue =
CALCULATE(
    SUM(Sales[Revenue]),
    Sales[Brand] = "Crocs",
    Sales[Channel] = "DTC"
)
```

---

### Time Intelligence — Know These Cold
Requires a proper Date table marked as a date table with no gaps.

```dax
-- Same period last year
Revenue LY =
CALCULATE(
    SUM(Sales[Revenue]),
    SAMEPERIODLASTYEAR(Dates[Date])
)

-- YoY % change
Revenue YoY % =
DIVIDE(
    [Total Revenue] - [Revenue LY],
    [Revenue LY],
    0   -- return 0 if denominator is blank/zero, not an error
)

-- Year-to-date
Revenue YTD =
TOTALYTD(
    SUM(Sales[Revenue]),
    Dates[Date]
)

-- Rolling 3 months
Revenue 3M Rolling =
CALCULATE(
    SUM(Sales[Revenue]),
    DATESINPERIOD(Dates[Date], LASTDATE(Dates[Date]), -3, MONTH)
)

-- Month-to-date
Revenue MTD = TOTALMTD(SUM(Sales[Revenue]), Dates[Date])
```

---

### Iterator Functions (X functions)
Used when you need to evaluate an expression row-by-row before aggregating. Common in retail for margin calculations.

```dax
-- SUMX: multiply two columns, then sum (can't do this with SUM alone)
Total Gross Margin =
SUMX(
    Sales,
    Sales[Revenue] - Sales[COGS]
)

-- AVERAGEX: weighted average (e.g., avg selling price weighted by units)
Avg Selling Price =
DIVIDE(
    SUMX(Sales, Sales[Revenue]),
    SUM(Sales[Units])
)

-- RANKX: rank products by revenue within brand
Product Revenue Rank =
RANKX(
    ALLSELECTED(Products[Product Name]),
    [Total Revenue],
    ,
    DESC,
    Dense
)
```

---

### RELATED & RELATEDTABLE
Used to reach across relationships in the model.

```dax
-- RELATED: pull a value from the "one" side into the "many" side (in calculated column)
Product Category = RELATED(Products[Category])

-- RELATEDTABLE: reference rows from the "many" side from the "one" side
Product Order Count =
COUNTROWS(RELATEDTABLE(Sales))
```

---

### ALL, ALLEXCEPT, ALLSELECTED

```dax
-- ALL: removes all filters from a table or column
Brand Revenue Share =
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALL(Sales[Brand]))
)

-- ALLEXCEPT: removes all filters EXCEPT specified columns
Revenue vs Brand Total =
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALLEXCEPT(Sales, Sales[Brand]))
)

-- ALLSELECTED: respects slicer selections but ignores visual-level filters
-- Useful for % of filtered total
Revenue % of Selection =
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALLSELECTED(Sales))
)
```

---

### VAR / RETURN — Always Use for Readable, Efficient DAX

```dax
Revenue YoY % =
VAR current_rev = [Total Revenue]
VAR prior_rev = [Revenue LY]
VAR yoy = DIVIDE(current_rev - prior_rev, prior_rev, 0)
RETURN
    IF(ISBLANK(prior_rev), BLANK(), yoy)
```

VAR evaluates once — avoids recalculating the same expression multiple times inside a measure.

---

### SWITCH — Cleaner than Nested IF

```dax
Channel Label =
SWITCH(
    Sales[Channel],
    "DTC", "Direct to Consumer",
    "WHS", "Wholesale",
    "Other"  -- default
)

-- SWITCH TRUE pattern for range logic
Performance Band =
SWITCH(
    TRUE(),
    [Revenue YoY %] >= 0.10,  "Strong Growth",
    [Revenue YoY %] >= 0,     "Flat to Growth",
    [Revenue YoY %] >= -0.10, "Moderate Decline",
    "Significant Decline"
)
```

---

## Power BI — Architecture & Optimization

### Star Schema vs Snowflake Schema
**Always use star schema in Power BI.** One fact table, flat dimension tables. Do not replicate normalized database structures in your model.

- Fact table: Sales, Orders, Inventory (numeric, high-row-count)
- Dimension tables: Products, Dates, Stores, Channels, Geography (descriptive, low-row-count)
- Relationships: many-to-one from fact to dimension, single direction by default

**Why it matters for performance:** Power BI's VertiPaq engine compresses columnar data — normalized joins across many tables kill performance and create ambiguous filter paths.

---

### Import vs DirectQuery vs Composite

| Mode | When to Use | Tradeoff |
|---|---|---|
| **Import** | Default. Data cached in memory. | Fastest query speed; data is only as fresh as last refresh. |
| **DirectQuery** | Real-time data required; data too large to import. | Every visual fires a live query — can be slow; limited DAX support. |
| **Composite** | Mix: import dimensions, DirectQuery fact table. | Flexible but complex; easy to create performance traps. |

For Crocs: most BI reporting will be Import mode with scheduled refresh. DirectQuery is rarely the right answer unless explicitly required.

---

### Semantic Model Optimization Checklist
This is the core of the Sr. role JD — know this deeply.

1. **Remove unused columns** — every column in Import mode is compressed into memory. Kill columns you don't use in visuals or DAX.
2. **Correct data types** — integers compress far better than text. Don't leave numeric IDs as text.
3. **Avoid calculated columns** — use measures instead. Calculated columns are computed at refresh and stored in memory; measures compute on demand.
4. **Hide technical columns** — fields used only in DAX relationships shouldn't be visible to report authors.
5. **Use a dedicated Date table** — never use auto date/time (it creates hidden tables that bloat the model). Mark it as a date table.
6. **Reduce cardinality** — high-cardinality text columns (like full product descriptions) compress poorly. Consider whether you need them in the model at all.
7. **Aggregation tables** — for large fact tables, create pre-aggregated summary tables (e.g., daily brand revenue) and configure Power BI to use them automatically for coarse-grain visuals.

---

### Deployment Pipelines
Power BI's built-in Dev → Test → Production promotion workflow (requires Premium or PPU).

- Each pipeline stage has its own workspace
- Promote content between stages with a single button click
- Can configure dataset rules (change connection strings per stage — dev Snowflake vs prod Snowflake)
- Supports selective deployment — push only changed items

**If asked about this (it's a noted gap):**
> "I've managed dev/prod promotion manually — publishing to different workspaces with different connection configs. I've been getting up to speed on the Deployment Pipeline feature specifically and understand the stage-based workflow and dataset rules. It's on my short list to get hands-on with in a sandbox."

---

### Tabular Editor
External tool for editing Power BI semantic models at the XMLA endpoint. Two main uses:

1. **Best Practice Analyzer** — runs rules against your model to catch issues (unused columns, missing descriptions, measure formatting)
2. **Scripting model changes** — C# scripts to apply bulk changes (rename, reformat, batch-create measures)
3. **Advanced model editing** — perspective management, calculation groups

**If asked about this (it's a noted gap):**
> "I haven't used Tabular Editor in production yet — it wasn't part of the toolchain at T-Mobile. I know its primary use cases: BPA for model governance and XMLA-based scripting for bulk changes. I'd treat it as a force multiplier on model maintenance work I already know how to do manually."

---

### Row-Level Security (RLS)
Restricts data visible to specific users within a report.

```dax
-- Static RLS role filter example: restrict to a brand
[Brand] = "HEYDUDE"

-- Dynamic RLS: filter to the logged-in user's region
[Region] = LOOKUPVALUE(
    UserMapping[Region],
    UserMapping[Email], USERPRINCIPALNAME()
)
```

Test RLS in Power BI Desktop using "View as role" before publishing.

---

### Performance Analyzer
Built into Power BI Desktop (View → Performance Analyzer). Records:
- DAX query time
- Visual display time
- Other (rendering)

High "DAX query" time = optimize the measure or model.
High "Other" time = too many visuals, too complex a report page.

**Also use:** DAX Studio (external) to capture and explain actual DAX queries generated by visuals.

---

### Report Design Principles (may come up in technical discussion)
- One clear question per page — don't pack everything onto one canvas
- Drill-through over drill-down when dimensions differ
- Bookmarks for toggling views (saves report state, not re-querying)
- Tooltips pages for contextual detail without cluttering the main view
- Don't use too many card visuals — they're expensive (each fires a separate query)

---

## DAX Practice Problems — Do These Out Loud

1. *"Write a measure that calculates revenue for the current selected period vs the same period last year, and returns BLANK if there's no prior year data."*

2. *"Write a measure that shows each brand's % share of total enterprise revenue, while respecting all active slicers except the brand slicer."*

3. *"A product manager wants to see a 'running total of DTC revenue by month that resets at the start of each year.' Write the measure."*

4. *"You have a Sales fact table and a Products dimension. Write a measure that returns the top 5 products by revenue for the current filter context."*

5. *"Your report has 15 card visuals at the top of a page and the page is loading in 8 seconds. Walk me through how you'd diagnose and fix it."*
   - Open Performance Analyzer — identify which visuals are slow
   - Check if cards are using complex measures — simplify or pre-aggregate
   - Consider replacing individual cards with a single matrix or table visual
   - Check model: are the measures hitting full-table scans?
   - Use DAX Studio to inspect the actual query

6. *"What's the difference between ALLSELECTED and ALL? When would you use each?"*
   - ALL removes ALL filters (including slicers) — use for grand totals
   - ALLSELECTED removes visual-level filters but respects slicers — use for "% of what the user has selected"
