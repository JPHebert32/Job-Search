# Crocs Final Round — Day-Of Cheat Sheet
*Keep this open. Read it in the car.*

---

## The Three Things You MUST Land

**1. Greenfield is your superpower.**
You built the T-Life HEART framework from zero — 30 dashboards, sole owner, 2 years. You've done this exact thing. Lead with enthusiasm, not just competence.

**2. You know why the last person was fired — and you're the answer to that.**
Silent data truncation. You caught it before leadership did. You built automated sanity checks. You documented it. That's the opposite of what got them fired. Make them feel safe.

**3. You'll listen first, build second.**
30/60/90: first 30 days = no dashboards, just stakeholder interviews and data audit. That's not delay — that's professionalism. It's what earns trust on a greenfield build.

---

## Data Integrity STAR (Have This Cold)

**S:** Sole owner of T-Life dashboards, 100+ daily visits from VPs. One upstream Snowflake table had a silent 500K-row COPY INTO truncation. Dashboard showing correct-looking numbers that were 15–20% understated.

**T:** Catch it before leadership does. Build a process so it can't happen silently again.

**A:** Built daily row-count sanity check — compared yesterday's event volume to 30-day rolling average, flagged >15% drop. Added data freshness timestamp to dashboard. Escalated to data engineering (fixed in 2 days). Applied correction factor to historical data. Formalized as standard: every report gets an "expected range" and automated alert.

**R:** Engineering patched the pipeline. Sanity check caught 2 more anomalies in the next 6 months — both before any stakeholder meeting. Stakeholders stopped finding problems before I did.

---

## 30/60/90 — Key Phrases

**Days 1–30 (Listen):** Stakeholder interviews — CHRO, HR BPs, VP Wholesale, key account managers. Data audit — what's in Snowflake, what's in the ERP, what HR system (Workday/ADP), what EDI/POS feeds exist. Define 3–5 most critical questions per team. *No dashboards yet.*

**Days 30–60 (First Deliverables):** HR → headcount by department + rolling attrition rate. Wholesale → sell-through by account + inventory aging. V1 as single-page reports. Get feedback. Write down every metric definition — name, formula, source, owner.

**Days 60–90 (Expand):** Full reporting suite. Row-level security on HR (managers see their org). Automated data freshness alerts. Document everything.

**Why this wins:** Sounds exactly like what they need after a data integrity failure. Thoughtful, not hasty.

---

## Quick Domain Anchors

**HR:** Headcount, attrition (voluntary vs involuntary), org hierarchy (recursive CTE), time-to-fill, comp bands. RLS is non-negotiable for HR data.

**Wholesale:** Sell-through rate (POS units sold / units shipped to retailer), inventory aging, door count, channel mix (wholesale vs DTC). Data usually lives across ERP + EDI feeds — reconciliation is the first question to ask.

---

## 5 Questions — Pick 3 That Feel Right In The Room

1. "Given the prior data integrity challenges — what would a successful first 90 days look like to you? What would give you confidence?"
2. "HR and Wholesale are greenfield — what are the 2–3 reports that would have the most immediate business impact at the 6-month mark?"
3. "What does the relationship between BI and data engineering look like? Clear data ownership handoffs, or still evolving?"
4. "How mature is the Power BI semantic model layer — shared certified datasets, or analysts building independently?"
5. "What's the biggest thing you're hoping this person brings that the team doesn't currently have?"

---

## Anchor Story Map (Quick Reference)

| They Ask About | Your Story |
|---|---|
| Data integrity / trust | T-Life Snowflake truncation + row-count sanity checks |
| Build from scratch | T-Life HEART framework — 30 dashboards, sole owner, 2 years |
| Stakeholder disagreement | Definitions tab, metric pushback at T-Life |
| Training / mentoring | T-Life 6-month knowledge transfer |
| Prioritization | Multiple product teams at T-Life competing for dashboard work |
| New domain | Network → SyncUP → T-Life → T-Sentiment (lateral moves within T-Mobile) |
| Automation / efficiency | T-Sentiment daily VP brief — fully automated, zero manual assembly |

---

## One-Liner Close If They Ask "Why Crocs?"

> "Honestly — the combination of a greenfield build in two genuinely interesting domains and a team that clearly cares about doing it right this time. That's a rare combination, and it's exactly the kind of work where I've done my best."
