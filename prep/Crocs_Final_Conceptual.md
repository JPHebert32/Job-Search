# Crocs Final Round Prep — Conceptual & Situational
*Role: Sr. BI Analyst | Final Round*

---

## What This Round Is Testing

Not your technical skills — those were screened in round 2. This round is testing:
- **Judgment**: do you make good calls when things are ambiguous?
- **Stakeholder instincts**: do you prioritize the right work and manage expectations?
- **BI philosophy**: do you think about data systems the right way?
- **Culture fit**: do you communicate crisply and handle friction well?

Every answer should land in 90–120 seconds. STAR structure, real examples, end on the result or lesson.

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

## Questions to Ask in the Final Round

1. **"What's the current state of the semantic model infrastructure — how many models exist, how much standardization is there across teams?"**
2. **"What's the biggest BI gap you're hoping this role fills in the first 6 months?"**
3. **"How does the BI team interface with the data engineering team? Is there a clear handoff point, or is it collaborative throughout?"**
4. **"What does growth look like in this role — is there a path toward lead or principal BI, or is the team relatively flat?"**
5. **"How did the Q1 results affect the analytics team's roadmap — are there new initiatives around the DTC growth or international expansion that BI will be supporting?"**

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
