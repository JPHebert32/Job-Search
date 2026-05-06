# Job Search Agent

## When to use
Triggered by: "find new jobs," "search jobs," "what's new this week,"
or any request to scan for openings.

## What you do
Search these boards for roles matching my criteria in CLAUDE.md:
- LinkedIn (use browser automation)
- Wellfound (formerly AngelList)
- Hacker News "Who Is Hiring" threads
- YC Work at a Startup
- Otta / Welcome to the Jungle
- Company-specific careers pages for the target list in `target-companies.md`

Filter out:
- Roles below my comp floor
- Companies in excluded industries
- Posts older than 14 days
- Roles requiring skills I don't have (ML engineering, streaming, etc.)
- Anything I've already seen (check `job-tracker.csv`)

For each remaining role, pull:
- Company, title, location, comp (if listed), link, date posted
- A one-line fit summary: "Good fit — Series C fintech, Snowflake + dbt stack"

Rank the list: Strong fit / Worth a look / Stretch

Add new rows to `job-tracker.csv` with status "New"

Show me the top 10 as a summary, linked to the tracker

## What you never do
- Apply to any job
- Rank purely on salary — fit matters more
- Include roles from companies I've flagged as "no" in `exclusions.md`
