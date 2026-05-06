# Job Search Agent

## When to use
Triggered by: "find new jobs," "search jobs," "what's new this week," morning/midday brief, or any request to scan for openings.

Run **twice daily**: morning (8–9 AM MDT) and midday (12–1 PM MDT). Evening sweep optional if pipeline is thin.

---

## Step 1 — Search these boards in order

### PRIMARY (search every run)

**Built In Colorado** — best signal-to-noise for Denver/Boulder tech
- https://www.builtincolorado.com/jobs/data-analytics/business-intelligence
- https://www.builtincolorado.com/jobs/data-analytics/business-intelligence/senior
- https://www.builtincolorado.com/jobs/remote/data-analytics/business-intelligence

**LinkedIn**
- Search: `"senior BI analyst" OR "senior business intelligence analyst" OR "lead BI analyst" OR "senior power BI"` — Location: Denver/Remote — Posted: Last 7 days
- Search: `"power BI" "senior data analyst"` — Remote US — Salary filter: $115K+ — Posted: Last 7 days
- https://www.linkedin.com/jobs/microsoft-power-bi-jobs-denver-metropolitan-area

**Wellfound (AngelList)**
- https://wellfound.com/role/r/business-intelligence
- https://wellfound.com/role/r/power-bi-developer
- Filter: Remote, $100K+, Series B through IPO

**Hacker News — Who Is Hiring (current month)**
- https://news.ycombinator.com/item?id=47975571 (May 2026)
- Use HNHIRING aggregator: https://hnhiring.com/locations/remote — search "business intelligence" and "power BI"

**YC Work at a Startup**
- https://www.workatastartup.com/jobs?q=business+intelligence&remote=true

### SECONDARY (search every other run / when pipeline is thin)

**Glassdoor**
- https://www.glassdoor.com/Job/denver-business-intelligence-analyst-jobs-SRCH_IL.0,6_IC1148170_KO7,36.htm
- https://www.glassdoor.com/Job/remote-power-bi-jobs-SRCH_IL.0,6_IS11047_KO7,15.htm

**Indeed**
- https://www.indeed.com/q-senior-business-intelligence-analyst-l-denver,-co-jobs.html
- https://www.indeed.com/q-power-BI-data-analyst-remote-jobs.html

**Dice.com** (good for Power BI / data roles)
- https://www.dice.com/jobs/q-remote+bi+analyst-jobs

**Remote-specific boards**
- https://arc.dev/remote-jobs/power-bi
- https://www.flexjobs.com/remote-jobs/business-intelligence
- https://dailyremote.com/remote-power-bi-jobs

### TIER 1 TARGET COMPANY CAREERS PAGES (check every run)

Scan each directly — do NOT rely on job boards to surface these:
- Pax8: https://www.pax8.com/en-us/careers/
- Ibotta: https://ibotta.com/careers
- Guild Education: https://www.guildeducation.com/careers/
- Homebot: https://www.homebot.ai/careers
- SambaSafety: https://sambasafety.com/careers/
- Checkr: https://checkr.com/company/careers
- Navan: https://navan.com/careers
- Ramp: https://ramp.com/careers
- Modern Health: https://www.modernhealth.com/careers
- Spring Health: https://www.springhealth.com/about/careers

See `target-companies.md` for the full list including Tier 2 and 3.

---

## Step 2 — Title variations to search (all of these, every run)

- Senior Business Intelligence Analyst
- Senior BI Analyst
- Lead Business Intelligence Analyst
- Lead BI Analyst
- Senior BI Developer
- Senior Power BI Developer
- Senior Power BI Analyst
- Senior Data Visualization Analyst
- Senior Business Intelligence Developer
- Senior Analytics Engineer (stretch — only if Power BI + dashboard work explicitly mentioned)
- Senior Product Analyst (stretch — only if dashboard ownership + SQL are primary duties)

---

## Step 3 — Filter criteria (hard stops — eliminate before scoring)

| Filter | Rule |
|--------|------|
| Comp | Eliminate if max of listed range < $115K. If comp not listed, do NOT eliminate — score and flag. |
| Industry | Eliminate: crypto, gambling, defense, ad-tech, marketing agencies |
| Location | Keep: remote US, Denver Metro (within ~40 min of Littleton), hybrid ≤ 3 days/week |
| Stack | Eliminate only if posting explicitly requires ONLY Domo or ONLY Tableau AND excludes Power BI. Metabase, Looker acceptable. |
| Posted | Eliminate if posted > 14 days ago |
| Already tracked | Check `job-tracker.csv` — skip if company + role already has any status except "Rejected" or "Ghosted" |
| Stage | Eliminate seed-stage (<Series A) and pure F500 enterprise (Fortune 100) |

---

## Step 4 — Score every remaining lead (1–10)

Score each lead before including it in the brief. Add score to the output table.

| Dimension | Max Points | Scoring Notes |
|-----------|-----------|---------------|
| **Stack match** | 3 pts | Power BI explicit = 3; Power BI + Snowflake/Fabric = 3; Tableau/Looker acceptable = 2; no tool listed = 1 |
| **Comp range** | 2 pts | Floor ≥ $130K = 2; floor $115–$129K = 1; unlisted = 1; below floor = 0 (eliminated) |
| **Company stage/type** | 2 pts | Series B–D or pre-IPO SaaS/fintech/consumer = 2; public mid-cap tech = 1; PE-owned or F500 = 0 |
| **Location fit** | 1 pt | Remote US = 1; Denver Metro hybrid ≤ 3 days = 1; hybrid >3 days = 0 |
| **Role ownership level** | 1 pt | "Own," "lead," or "sole" BI function = 1; team member within large org = 0 |
| **Industry fit** | 1 pt | B2B SaaS, fintech, consumer tech, marketplaces = 1; healthcare, legal, traditional industries = 0 |

**Threshold for inclusion in brief:**
- Score 7–10 = **Strong fit** ✅ — include with "apply today" recommendation
- Score 5–6 = **Worth a look** ⚠️ — include with caveats
- Score 1–4 = **Weak fit** — log elimination reason, do not include

---

## Step 5 — Output format

Add new rows to `job-tracker.csv` with status "New" for anything score ≥ 5.

Show in the daily brief as a table:

| Score | Company | Role | Comp | Location | Posted | Link | Fit |
|-------|---------|------|------|----------|--------|------|-----|
| 8/10 | PopSockets | BI Lead | $93K–$114K (est) | Boulder CO | May 5 | [link] | ✅ |

Then show:
1. Fit notes for each ✅ and ⚠️ lead (2–3 sentences: why it fits, what to verify, stack alignment)
2. Screened-out leads with one-line reason each
3. Target company page status: which were checked, what was found (even if nothing new)

---

## Step 6 — Pipeline health check (run at morning brief)

Report this at the top of each morning brief:

```
Pipeline summary:
- Active applications: X (status: Applied or In Process)
- Ready to Apply (stale > 5 days): X — flag each by name
- New leads this week: X
- Interviews active: X
```

If Active applications + Ready to Apply < 5 total: **flag as pipeline risk** and trigger an aggressive search run (secondary boards + all target company pages).

---

## What you never do
- Apply to any job
- Rank purely on salary — fit matters more
- Include roles from excluded industries in `exclusions.md`
- Include roles already in `job-tracker.csv` with status other than Rejected/Ghosted
- Add rows to tracker without JP's review of the brief first
