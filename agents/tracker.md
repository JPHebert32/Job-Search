# Application Tracker

## When to use
- Every time I apply, update the corresponding row in the tracker
- When I get an email response (recruiter reply, rejection, interview invite), update the status
- When I ask "where am I with [company]" or "what's my pipeline"

## Structure of job-tracker.csv
Columns: `Date Added | Company | Role | Location | Comp (if known) | Link | Status | Last Action | Next Step | Next Step Due | Notes`

## Status values
- **New** — found but not yet reviewed
- **Reviewing** — I'm considering it
- **Applied** — submitted
- **Screen Scheduled**
- **Screen Done**
- **Take-Home In Progress**
- **Take-Home Submitted**
- **Onsite Scheduled**
- **Onsite Done**
- **Offer**
- **Rejected**
- **Withdrawn**
- **Ghosted** (no response 21+ days)

## What you do
- When I apply to a role, move it to "Applied" and set Next Step Due = +7 days for a follow-up
- When a recruiter email comes in, parse it and update the row
- Every Monday morning, show me:
  - Active pipeline (everything not Rejected/Withdrawn/Ghosted)
  - Anything stalled (no action in 10+ days) — suggest follow-ups
  - Next 7 days of interviews / take-homes / deadlines

## Rules
- Never close a row to Ghosted without showing me first
- Don't auto-send follow-ups — draft them and let me review
