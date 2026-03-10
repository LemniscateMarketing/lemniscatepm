---
template_id: client-onboarding
title: Client Onboarding
category: operations
estimated_total_hours: 12
task_count: 7
default_duration_weeks: 2
---

## When to Use

Apply this template when:
- A new client has signed a contract and onboarding begins
- A returning client is starting a new engagement or retainer
- A dormant account is being reactivated

Apply it the day the contract is signed so nothing falls through the cracks.

## Prerequisites

Before applying this template, confirm you have:
- Signed contract or SOW
- Client primary contact name and email
- Client billing contact (if different)
- Kickoff call date agreed
- Scope of work agreed (services, deliverables, cadence)

## Task Order and Dependencies

| # | Task | Hours | Depends On |
|---|---|---|---|
| 1 | contract-and-billing | 1h | — |
| 2 | access-collection | 2h | — |
| 3 | kickoff-prep | 2h | — |
| 4 | kickoff-call | 1h | 2, 3 |
| 5 | project-setup | 2h | 4 |
| 6 | welcome-pack | 2h | 4 |
| 7 | 30-day-check-in | 1h | 5, 6 |

**Note:** Tasks 1–3 can all start simultaneously. Tasks 4+ follow in order.
**Minimum viable duration:** 7 days to kickoff, 30 days to check-in.

## PLACEHOLDER Values

| Placeholder | Replace With |
|---|---|
| `PLACEHOLDER_CLIENT` | client slug (e.g. `acme-corp`) |
| `PLACEHOLDER_CLIENT_NAME` | Full client company name |
| `PLACEHOLDER_CONTACT` | Primary contact name and email |
| `PLACEHOLDER_ASSIGNEE` | @handle from TEAM.md |
| `PLACEHOLDER_PROJECT` | project ID from projects/ folder |

## AI Guidance

When a user says "apply the client onboarding template", do the following:

1. **Ask for missing info** (if not already provided):
   - "What's the client's name and their primary contact?"
   - "When is the kickoff call?"
   - "Who is the account lead / assignee?"

2. **Create the project file** in `projects/` first (if it doesn't exist):
   - Use format: `project-{client-slug}.md`
   - Set status: active

3. **Create all 7 task files** in `tasks/`:
   - Generate a fresh ID for each: `Date.now().toString(36).slice(-6)`
   - Replace all PLACEHOLDER values
   - Wire dependency IDs
   - Due dates: space tasks 2–3 days apart from kickoff date backwards and forwards

4. **Update manifest.json** with all new entries.

5. **Report back:**
   - List of tasks created with dates
   - Reminder: "Task 7 (30-day check-in) is due on [date] — it will not send automatically"
   - Any missing info to collect before kickoff

## Notes for the Assignee

The first 48 hours of a client relationship set the tone. Prioritise speed on
contract and access collection. Never let a client wait more than 1 business day
for their first touchpoint after signing.

---

_Template version: 1.0 — Created 2026-03-10 by @claude_
_Part of Loop workspace for Lemniscate Marketing_
