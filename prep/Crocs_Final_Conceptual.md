# Crocs Final Round Prep — Conceptual & Situational
*Role: Sr. BI Analyst | Final Round*

---

## What This Round Is Testing

Not your technical skills — those were screened in round 2. This round is testing:
- **Judgment**: do you make good calls when things are ambiguous?
- **Stakeholder instincts**: do you prioritize the right work and manage expectations?
- **BI philosophy**: do you think about data systems the right way?
- **Culture fit**: do you communicate crisply and handle friction well?
- **Data integrity specifically**: the previous analyst was let go over data integrity issues — you WILL get a question in this category. Have your best story ready cold.

Every answer should land in 90–120 seconds. STAR structure, real examples, end on the result or lesson.

---

## PRIORITY #1: The Greenfield Opportunity — Lead With This

**What you now know:** HR and Wholesale reporting does not exist at Crocs. You are walking into a blank canvas. No legacy reports to untangle, no inherited metrics you don't trust, no previous analyst's logic you have to reverse-engineer before you can improve it.

This is your strongest framing for the final round. You've done exactly this before — T-Life HEART framework was built from scratch, as the sole BI analyst, for a 75M-download app. That's the proof point.

**How to use this in the interview:**

When they describe the scope (or when you confirm it with a question), respond with genuine enthusiasm, not just calm acknowledgment:

> "Honestly, a greenfield build is where I do my best work. When I joined the T-Life team, there was no established BI framework for the product — I designed the entire KPI structure, the HEART framework, from the ground up. Thirty dashboards, sole ownership, two years. What I've learned is that the first 90 days of a greenfield build are the most important — they set the data model, the metric definitions, and the stakeholder trust that everything else runs on. I'd rather start clean than inherit something I can't fully stand behind."

**Your 30/60/90 for Crocs — be ready to walk through this:**

*Days 1–30 — Discovery*
- Stakeholder interviews: CHRO and HR BPs for HR domain; VP Wholesale and key account managers for Wholesale
- Data audit: understand what lives in Snowflake, what's in the ERP, what HR system Crocs uses (Workday? ADP?), and what EDI/POS feeds exist for Wholesale
- Define the 3–5 most critical questions each team needs answered — not a full backlog, just the highest-value starting point
- No dashboards built yet — this is listening time

*Days 30–60 — First Deliverables*
- HR: headcount by department + rolling attrition rate — the two metrics every CHRO wants first
- Wholesale: sell-through by account + inventory aging — the two metrics that drive the most urgent decisions
- Ship v1 of each as a single-page report; get stakeholder feedback before building out
- Establish data definitions in writing — every metric gets a name, formula, source table, and owner

*Days 60–90 — Expand and Standardize*
- Build out the full reporting suite based on feedback from v1
- Implement row-level security for HR (managers see their org, not others)
- Set up data freshness monitoring — automated alerts if source data stops refreshing
- Document everything: data dictionary, model guide, refresh schedules

**Why this story wins:** The previous analyst was let go over data integrity. The hiring team is acutely aware of what happens when someone builds fast and loose. Your 30/60/90 shows you do the opposite — you listen first, define clearly, build carefully, and document as you go.

---

## PRIORITY #2: Data Integrity — Your Must-Have Story

This is no longer a generic BI question. The Director told you the previous analyst was let go because of data integrity problems. That means the hiring team is actively looking for someone who will *not* repeat that failure. This question is coming. Prepare it like it's question #1.

**"Tell me about a time you caught or prevented a serious data quality issue."**

> **Situation:** At T-Mobile, I was the sole owner of the T-Life HEART dashboard suite — 30 reports used daily by product managers, and at peak, over 100 dashboard visits per day including VP-level. The data feeding those reports came from multiple Snowflake tables with different refresh cadences. Early in my ownership, I discovered that one upstream table — tracking user engagement events — had a silent truncation issue: it was only loading the first 500K rows of each day's event feed, silently dropping the rest. The dashboard was showing correct-looking numbers that were actually 15–20% understated.

> **Task:** I needed to catch this before leadership did and establish a process so it could never happen again silently.

> **Action:** I built a daily row-count sanity check — a simple query that compared yesterday's event volume to a rolling 30-day average and flagged anything more than 15% below baseline. I also added a visible "data freshness" indicator to the dashboard showing the last successful load time. I escalated the root cause to the data engineering team, which turned out to be a Snowflake COPY INTO statement with a default row limit that nobody had noticed. While they fixed the upstream issue, I applied a correction factor to historical reporting and documented the period of affected data.

> **Result:** The engineering team patched the pipeline within two days. More importantly, the sanity check I built caught two additional data anomalies over the next six months — both before they ever surfaced in a stakeholder meeting. I also formalized that pattern into a standards document: every report I owned had a defined "expected range" for its key metrics, and any automated refresh outside that range triggered an alert.

*Why this story works here:* It shows proactive detection (not reactive), systematic resolution, stakeholder protection, and a process left behind — exactly the opposite of what got the last person fired.

---

**"How do you ensure the numbers in your reports are trustworthy?"**

This is the same question, softer framing. Answer with your three-layer approach, then anchor it to the above story.

> "Three layers. First, source validation — I build row count checks, null checks, and range assertions into my ETL monitoring so data anomalies surface before they hit Power BI. Second, visual sanity checks — I keep a mental model of what 'normal' looks like for each KPI, so a number that's off by an order of magnitude triggers my gut before any automated check does. Third, trust-building with stakeholders — I put a data freshness timestamp and a definitions tab in every report I own, so users always know how current the data is and what each metric means. The combination means stakeholders rarely discover problems before I do, and when they do flag something, I already have the context to resolve it fast."

---

## Scenarios by Category

### Data Quality & Trust

**"A stakeholder says the numbers in your dashboard don't match what they calculated in Excel. How do you handle it?"**

This is one of the most common questions in BI roles. They want to see that you don't get defensive, you investigate systematically, and you communicate proactively.

> "First, I treat it as a legitimate data problem until proven otherwise — I don't assume the stakeholder is wrong. My first step is to ask them to share the Excel calculation and the specific time period so I can replicate it. Then I trace the discrepancy: is the dashboard filtering something they aren't? Is the date logic different — are they using invoice date vs ship date? Is there a currency conversion? Most discrepancies come down to definition misalignment, not a bug. Once I find the source, I document it, update any data definitions in the report, and often that conversation becomes the basis for a formal metric definition that prevents the same question from coming up again with the next person."

*Your T-Life hook:* At T-Mobile, I had product managers comparing their own query results to my HEART dashboards. The standard I implemented was a definitions tab in every report — each metric had a name, formula, source table, and logic notes. It cut those conversations by about 80%.

---

**"How do you ensure data quality in the reports you own?"**

> "Three layers. First, source validation — I build sanity checks into my Snowflake queries: row count checks, null checks on key columns, min/max range assertions. If something looks wrong at the source, I want to catch it before it surfaces in Power BI. Second, visual checks — I keep a 'known number' for key metrics in my head. If total revenue comes in at $50M instead of $500M, I need to catch that before leadership does. Third, stakeholder feedback loop — I treat every 'that looks wrong' message as signal, even if they turn out to be wrong. I keep a running log of data issues, their root causes, and resolutions. It builds trust over time."

---

### Prioritization & Competing Demands

**"You have three teams requesting new dashboards at the same time. How do you prioritize?"**

> "I ask the same questions for each request: What decision does this support? When does that decision need to be made? Is the data already available, or does this require new infrastructure? Can I deliver a lightweight version faster than the full ask? That framework usually breaks the tie. If two requests are genuinely equal, I default to the one with the highest organizational reach — a report the VP uses to make a quarterly call beats one a single analyst uses for a weekly check-in. I also communicate the queue explicitly — 'you're third, here's my estimated timeline' — so nobody is surprised."

---

**"How do you handle a business stakeholder who keeps changing requirements mid-build?"**

> "I try to front-load the requirement work so mid-build changes are smaller. Before I write a line of DAX, I build a mockup in PowerPoint or on a whiteboard — what does the finished report look like, what filters will exist, what's the grain of the data? I get sign-off on that before I build. When changes still come in mid-build — and they always do — I assess impact: does this change the model, or just the visual? Model changes are expensive; visual changes are cheap. I communicate that tradeoff honestly: 'this is a 30-minute change' vs 'this requires restructuring the semantic model, which will take a day and affect three other reports.' That usually helps stakeholders triage their own requests."

---

### Building From Scratch / New Analytics Domain

**"You're joining Crocs and the first thing leadership asks is for a dashboard that tracks HEYDUDE brand recovery. Where do you start?"**

This is custom to Crocs — it shows you've been paying attention.

> "I start with the business question: what does 'recovery' mean to leadership? Is it revenue, gross margin, wholesale door count, DTC conversion rate, or some combination? Those are different metrics that might even tell contradictory stories. My first two weeks would be stakeholder interviews — the HEYDUDE brand team, Planning, Finance — to understand how they define success and what decisions get made off this data.

> Concurrently, I'm doing a data audit: what's available in Snowflake, how fresh is it, what's the grain (daily? weekly? by SKU?), are there any known quality issues. Once I know the question and the data, I build a v1 model and a single-page report, walk it back to the stakeholders, and iterate from feedback rather than building a 10-page suite that nobody asked for.

> Given what I know about HEYDUDE publicly — the wholesale reset, the DTC pivot — I'd expect the core metrics to be: sell-through by channel, DTC acquisition and retention rates, wholesale door count vs target, and gross margin by channel mix."

---

**"How do you approach building a BI solution in a domain you don't know?"**

> "The domain is learnable — the analytical process isn't domain-specific. My approach: spend the first week listening more than building. I identify the three to five people who know the data best and the three to five people who make decisions off it, and I get time with both groups. The people who know the data will tell me about its quirks; the people who make decisions will tell me what questions matter. Then I prototype fast, show it early, and correct course. I don't wait until I have a perfect understanding of the domain to deliver value — I deliver incrementally and let the feedback teach me the domain faster than any amount of upfront study would."

---

### Mentoring & Standards

**"Describe how you've mentored or trained other BI developers or analysts."**

Use your T-Life knowledge transfer directly.

> "In my last 6 months at T-Mobile, my primary job was training the analysts who were taking over the T-Life dashboards. I had 30 reports across two suites, all built on models I'd developed and maintained for two years — I needed to transfer that knowledge completely.

> I built full documentation: a data dictionary for every table I used, a model guide explaining every relationship and why it was structured that way, and a report guide with the business logic behind each KPI. I also ran walkthroughs where I'd narrate my reasoning live while building a new report, so they could see the decision-making, not just the output.

> The standard I set for the handoff was: they should be able to modify any existing report and build a new one without calling me. After 6 months, they could. I also informally mentored product managers on how to read and interact with the dashboards — how filters worked, what the metrics meant — because a dashboard that isn't trusted or understood doesn't get used."

---

### BI Philosophy

**"What makes a great BI dashboard vs a mediocre one?"**

> "A great dashboard answers a specific question before the user has to ask it. It's designed around a decision flow, not a data structure. The mediocre ones are built for the analyst — they show everything available. The great ones are built for the decision-maker — they show what matters, surface anomalies automatically, and make the next action obvious.

> Technically, great dashboards also load fast, have clean data definitions, and don't surprise users with numbers they can't verify. Trust is slow to build and fast to lose. If leadership opens a dashboard and the numbers look wrong twice, they stop opening it."

---

**"How do you balance stakeholder requests with BI best practices when they conflict?"**

> "I try to understand the intent behind the request, not just the request itself. When someone asks for something that's technically problematic — a calculated column that should be a measure, a metric definition that's inconsistent with how the rest of the org defines it — I explain the tradeoff: 'Here's what you asked for, here's the downstream risk, here's an alternative that accomplishes the same goal.' I don't just say no, and I don't just comply. Most stakeholders don't care about DAX semantics — they care about getting the right answer. My job is to get them the right answer in a way that doesn't create technical debt."

---

### Culture & Fit (Broomfield HQ — Know the Context)

**"How do you handle ambiguity or changing priorities?"**

> "I've worked in consulting-embedded roles where my priorities shifted based on what T-Mobile leadership cared about that quarter. The practice I developed was to maintain a clear picture of what I owned and what my stakeholders' most time-sensitive needs were — so when new requests came in, I could quickly assess whether they displaced something on my list or were additive. I communicate proactively: 'I can take this on, but here's what moves. Is that the right tradeoff?' That usually works because it makes the prioritization decision explicit rather than invisible."

---

**"What's your approach when you disagree with how leadership wants data presented?"**

> "I present the concern once, clearly, with a specific alternative. Something like: 'I understand you want to show month-over-month growth here, but the baseline period had an anomaly that makes that number misleading — can we show a 3-month rolling average instead, or add a footnote explaining the baseline?' I make the recommendation, explain the risk of not taking it, and then defer to their decision if they still want the original approach. I document that the presentation decision was made by stakeholder request. I don't refuse to build something because I'd design it differently — but I make sure the record reflects accurate data, even if the framing is their call."

---

## HR & Wholesale Reporting Domain — Know the Territory

You now know the two primary reporting areas for this role. If the final round goes technical, these are the domains you'll be tested in. Even if it stays behavioral, you should speak fluently about them.

### HR Reporting — Core Concepts

HR reporting in BI typically covers:
- **Headcount** — active employee count by department, location, level, hire type (FTE vs contractor)
- **Attrition / Turnover** — voluntary vs involuntary, rolling 12-month rate, by department or manager
- **Org hierarchy** — recursive / self-referencing employee-manager tables; common pattern in Snowflake
- **Time-to-fill / Time-to-hire** — open requisition age, recruiter funnel metrics
- **Comp & Bands** — salary distribution by level/band, equity gap analysis
- **Headcount planning vs actuals** — comparing approved headcount to filled seats

**Key SQL pattern — recursive org hierarchy (DAX or SQL):**
```sql
-- Org hierarchy with recursive CTE
WITH RECURSIVE org_tree AS (
    SELECT employee_id, manager_id, name, 1 AS level
    FROM employees
    WHERE manager_id IS NULL  -- CEO / top of org

    UNION ALL

    SELECT e.employee_id, e.manager_id, e.name, ot.level + 1
    FROM employees e
    JOIN org_tree ot ON e.manager_id = ot.employee_id
)
SELECT * FROM org_tree ORDER BY level, manager_id;
```

**Your angle:** "HR data is sensitive — report-level row security matters. In Power BI I'd implement RLS so HR BPs only see their own orgs, and managers only see their direct reports unless they have elevated access."

---

### Wholesale Reporting — Core Concepts

Wholesale at a consumer brand like Crocs means selling through retail partners (Foot Locker, Dick's, Amazon, etc.) rather than direct-to-consumer. Key metrics:

- **Sell-through rate** — % of units sold to end consumers vs units shipped to retail partner (shipped ≠ sold)
- **Inventory aging** — how long units have been sitting in a retailer's warehouse; aged inventory = markdown risk
- **Door count** — number of retail locations carrying a product; expansion vs. contraction
- **Orders vs. shipments vs. POS** — three different stages with different data sources; mismatches are common
- **Channel mix** — wholesale vs DTC revenue split; Crocs has been pivoting DTC-heavy, so wholesale % is a watched metric
- **Return rates by account** — high returns from a specific retailer can signal a merchandising problem

**Key SQL pattern — sell-through by account:**
```sql
SELECT
    r.retail_account,
    p.product_line,
    SUM(s.units_shipped) AS units_shipped,
    SUM(pos.units_sold)  AS units_sold,
    ROUND(SUM(pos.units_sold) * 1.0 / NULLIF(SUM(s.units_shipped), 0), 3) AS sell_through_rate,
    SUM(s.units_shipped) - SUM(pos.units_sold) AS units_remaining
FROM shipments s
JOIN pos_data pos ON s.sku = pos.sku AND s.retail_account_id = pos.account_id
JOIN retailers r ON s.retail_account_id = r.id
JOIN products p ON s.sku = p.sku
WHERE s.ship_date >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY r.retail_account, p.product_line
ORDER BY sell_through_rate ASC;  -- surface worst performers first
```

**Your angle:** "Wholesale data often lives in multiple systems — ERP for orders/shipments, retailer EDI feeds or syndicated data (like SPS Commerce) for POS. One of my first steps in this role would be understanding where each source lives in Snowflake and whether POS and shipment data are being reconciled automatically or manually."

---

## Questions to Ask in the Final Round

These are calibrated for a final round — they signal that you're evaluating them as much as they're evaluating you, and that you're already thinking about how to succeed in the role.

1. **"Given that this role is stepping into a situation with prior data integrity challenges — what would a successful first 90 days look like to you? What would give you confidence that the right foundation is being laid?"**
   *(Shows you heard what the Director said and are thinking proactively about solving the actual problem.)*

2. **"I understand HR and Wholesale reporting is essentially a greenfield build — what does success look like at the 6-month mark? What are the 2–3 reports that would have the most immediate business impact?"**
   *(You already know it's greenfield — asking this shows you've done your homework and are already thinking about prioritization. Surfaces what leadership actually cares about first.)*

3. **"What does the relationship between the BI team and the data engineering team look like? Is there a clear handoff for data ownership, or is that still evolving?"**
   *(Important for understanding who you'll depend on for clean data.)*

4. **"How mature is the Power BI semantic model layer — are there shared certified datasets, or are analysts building their own models independently?"**
   *(Shows semantic model awareness and surfaces whether you're inheriting a mess or a foundation.)*

5. **"What's the biggest thing you're hoping this person brings that the team doesn't currently have?"**
   *(Direct but powerful — surfaces unstated priorities and lets you address them directly.)*

---

## Your Anchor Stories — Map to Questions

| Scenario Type | Your Story |
|---|---|
| Complex build from scratch | T-Life HEART framework — 30 dashboards, sole owner, 2 years |
| Stakeholder disagreement | Metric definition pushback at T-Life |
| Data discrepancy / trust issue | Metric definitions tab, T-Life standards |
| Training / mentoring | T-Life knowledge transfer, 6 months |
| Brand/sentiment analytics | T-Sentiment — daily VP briefs, cross-platform tracking |
| Prioritization under pressure | Multiple product teams at T-Life competing for dashboard work |
| New domain | Any lateral move within T-Mobile (Network → SyncUP → T-Life → Sentiment) |
