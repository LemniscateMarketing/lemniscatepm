---
template_id: seo-audit
title: SEO Audit
category: seo
estimated_total_hours: 32
task_count: 8
default_duration_weeks: 3
---

## When to Use

Apply this template when:
- Onboarding a new SEO client for the first time
- Running a periodic SEO health check for an existing client (quarterly recommended)
- Diagnosing a traffic drop or ranking decline
- Preparing a pitch or proposal that includes an SEO assessment

## Prerequisites

Before applying this template, confirm you have:
- Client website URL
- Access to Google Search Console (or client confirms it exists)
- Access to Google Analytics 4 (or UA if still active)
- List of 2–4 main competitors (ask the client if unknown)
- Project deadline from the client

## Task Order and Dependencies

Tasks are listed in recommended execution order. Dependencies are strictly enforced
— do not start a dependent task until all upstream tasks are complete.

| # | Task | Hours | Depends On |
|---|---|---|---|
| 1 | keyword-research | 8h | — |
| 2 | competitor-analysis | 6h | — |
| 3 | on-page-audit | 4h | 1 |
| 4 | technical-audit | 4h | — |
| 5 | backlink-audit | 4h | 1 |
| 6 | content-gap-analysis | 3h | 1, 2 |
| 7 | page-speed | 2h | — |
| 8 | final-report | 1h | 1, 2, 3, 4, 5, 6, 7 |

**Critical path:** keyword-research → on-page-audit → final-report
**Minimum viable duration:** 14 days (do not promise faster)

## PLACEHOLDER Values

When applying this template, replace all PLACEHOLDER values:

| Placeholder | Replace With |
|---|---|
| `PLACEHOLDER_CLIENT` | client slug (e.g. `acme-corp`) |
| `PLACEHOLDER_ASSIGNEE` | @handle from TEAM.md |
| `PLACEHOLDER_PROJECT` | project ID from projects/ folder |
| `PLACEHOLDER_DOMAIN` | client's website URL |

IDs are generated at apply-time using `Date.now().toString(36).slice(-6)`.
Dates are calculated backwards from the client deadline.

## AI Guidance

When a user says "apply the SEO audit template", do the following:

1. **Ask for missing info** (if not already provided):
   - "What's the client's website URL?"
   - "Who are their main 2–4 competitors?"
   - "What's the deadline?"
   - "Who is the assignee? (leave blank to assign to yourself)"
   - "Any tasks to skip? (e.g. backlink audit is optional for small sites)"

2. **Validate the deadline:**
   - If deadline < 14 days from today: warn the user and ask to confirm
   - Calculate backwards: final-report due = deadline, all others = deadline minus 1–4 days per task in reverse critical path order

3. **Create all 8 task files** in `tasks/` (or fewer if some tasks are skipped):
   - Generate a fresh ID for each: `Date.now().toString(36).slice(-6)`
   - Replace all PLACEHOLDER values
   - Wire dependency IDs using the newly generated IDs (not template placeholder names)
   - Set status: `todo` on all tasks
   - Set `project` if a project file exists for this client

4. **Update manifest.json** with all new task entries.

5. **Report back** with:
   - List of tasks created with IDs and due dates
   - Critical path summary
   - Any warnings (tight deadline, missing access, skipped tasks)

## Notes for the Assignee

SEO audits for new clients typically reveal 20–40 issues. Prioritize findings
by traffic impact, not volume. The final report should be written for a
non-technical client — avoid jargon, lead with business impact.

---

_Template version: 1.0 — Created 2026-03-10 by @claude_
_Part of Loop workspace for Lemniscate Marketing_
