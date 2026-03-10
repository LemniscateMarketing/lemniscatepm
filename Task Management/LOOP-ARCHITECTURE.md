# Loop — Product Architecture & Plan
> Version 3.1 — March 2026
> Status: Active review. AI reviewers please add notes in Section 22.

## Changelog
- v3.1: Incorporated Codex review feedback. ID length → 6 chars. manifest
  recovery made automatic. Conflict resolution downgraded to best-effort v1.
  Save-time validator added as 6th JS function. Dashboard internal-only label
  added. Replit reviewer slot added in Section 22.
- v3.0: Settled final architecture after three rounds of review. Replaced
  loop-core module with targeted JS functions. Cleaned up YAML decision.
  Added Codex review response (Section 21). Added AI reviewer notes section.
- v2.0: Added loop-core, manifest.json, missing schemas, AI execution model.
- v1.0: Initial architecture memo.

---

## 1. Vision

Loop is an AI-native project management system for non-technical teams —
marketing agencies, creative studios, and client-service businesses.

**The stack in one sentence:** GitHub stores the files. Simple HTML + JS
renders the board and handles a few deterministic rules. The Chrome extension
is the daily interface. AI reads SCHEMA.md and handles everything else through
conversation.

No SaaS. No backend server. No CLI. No npm for end users. Just files, a
browser extension, and conversation.

---

## 2. Target Audience

**Primary:** Marketing agencies and creative teams (5–30 people). Designers,
copywriters, strategists, SEO specialists, account managers. Non-technical.
Work across multiple clients and projects simultaneously.

**Secondary:** Small product teams and consultancies who want full data
ownership and AI-first workflows.

**Not targeting (v1):** Enterprise (100+ people), developer teams (Backlog.md
serves them), real-time collaborative editing (Google Docs serves that).

---

## 3. The Three-Layer Stack

```
┌─────────────────────────────────────────────────────────┐
│                    LAYER 1: INTERFACES                   │
│                                                          │
│   Chrome Extension         Dashboard (HTML + JS)         │
│   Primary daily UI         Read-only board for team      │
│   AI chat + task UI        No install, open in browser   │
└──────────┬─────────────────────────┬────────────────────┘
           │                         │
           ▼                         ▼
┌─────────────────────────────────────────────────────────┐
│              LAYER 2: SIMPLE JS LOGIC                    │
│         (lives inside Chrome extension + dashboard)      │
│                                                          │
│   • Timestamp-based ID generation                        │
│   • Recurring task trigger (if done + repeat → create)   │
│   • manifest.json sync after every write                 │
│   • GitHub 409 conflict detection                        │
│   • YAML frontmatter parse/write (using gray-matter)     │
│                                                          │
│   That's it. ~100 lines of JS total. Not a module.       │
│   Not a package. Functions inside the extension.         │
└──────────┬─────────────────────────┬────────────────────┘
           │                         │
           ▼                         ▼
┌─────────────────────────────────────────────────────────┐
│               LAYER 3: GITHUB (THE DATABASE)             │
│                                                          │
│   Files · Version history · Access control               │
│   Conflict detection via SHA · Audit trail               │
│   OAuth authentication · API (5000 req/hr per user)      │
└─────────────────────────────────────────────────────────┘
```

**The rule for what goes where:**

If a computer must get it right every time and it's simple → JS logic.
If it needs understanding, context, or judgment → AI + SCHEMA.md.
If it's data or history → GitHub.

---

## 4. Core Philosophy

**1. Files are the product.** Everything is a Markdown file. No proprietary
format. If Loop disappears tomorrow, all data is still there, readable by
any human or AI.

**2. Simple JS for reliability, AI for intelligence.** JS handles the small
number of things that must be deterministic. AI handles everything that
benefits from reasoning. Never ask AI to do what 10 lines of JS can do
reliably. Never write 500 lines of code for what SCHEMA.md can explain.

**3. GitHub is the database.** Version history, access control, conflict
detection — GitHub provides this for free.

**4. Zero installation for end users.** Install a Chrome extension. Connect
a GitHub repo. Done. No npm, no terminal, no Homebrew.

**5. Any AI must understand it.** Claude, Codex, Gemini, GPT — any AI that
reads SCHEMA.md first can fully operate Loop. The schema is the contract.

**6. Templates are the business value.** The file format is infrastructure.
Domain-specific templates for SEO, GA4, campaigns, client onboarding — that
is where agency value lives.

---

## 5. What We Deliberately Exclude

| Excluded | Reason |
|---|---|
| Standalone CLI | Requires install. The few deterministic things live in the extension as simple JS. |
| Local file sync (Dropbox/iCloud) | Creates conflicts. GitHub API is the only write path. |
| Custom backend server | GitHub IS the database. No server needed. |
| Real-time co-editing | Not a Google Docs replacement. Tasks have one owner at a time. |
| Mobile app (v1) | Chrome extension covers 90% of use. Mobile is v3. |
| Proprietary file format | Plain Markdown forever. |
| Full validation layer | SCHEMA.md + AI handles this. Wrong priority label won't lose money. |
| Client-facing dashboard (v1) | Auth is complex to do safely. Internal team only for v1. |

---

## 6. Repository File Structure

```
loop-repo/
│
├── SCHEMA.md                     ← AI reads this first, every time
├── AGENTS.md                     ← Points AI to SCHEMA.md, brief orientation
├── TEAM.md                       ← Team members, roles, capacity
├── config.yml                    ← Project settings, timezone, DoD
├── dashboard.html                ← Self-contained HTML+JS board (read-only)
│
├── tasks/
│   ├── manifest.json             ← Lightweight index of all tasks
│   ├── task-lx4k-seo-audit-homepage.md
│   ├── task-m2p9-ga4-setup.md
│   └── task-n7q3-onboard-client-acme.md
│
├── projects/
│   ├── project-acme-rebrand.md
│   └── project-website-launch.md
│
├── templates/
│   ├── seo-audit/
│   │   ├── TEMPLATE.md           ← metadata + AI guidance for this template
│   │   ├── task-01-keyword-research.md
│   │   ├── task-02-competitor-analysis.md
│   │   ├── task-03-on-page-audit.md
│   │   ├── task-04-technical-audit.md
│   │   └── task-05-final-report.md
│   ├── ga4-setup/
│   ├── campaign-launch/
│   ├── client-onboarding/
│   └── content-calendar/
│
├── decisions/
│   └── 2026-03-10-chose-github-over-notion.md
│
├── docs/
│   └── client-brief-acme.md
│
└── attachments/
    ├── task-lx4k/
    │   ├── current-rankings.pdf
    │   └── competitor-analysis.xlsx
    └── task-m2p9/
        └── ga4-guide.pdf
```

---

## 7. Task ID System

**Not auto-increment** (collision under concurrent writes).
**Not ULIDs** (26 chars, ugly for non-technical users browsing files).
**Timestamp-based short ID** — generated by one line of JS.

```javascript
const id = Date.now().toString(36).slice(-6) // e.g. "lx4k9m"
```

`Date.now()` returns milliseconds since epoch. `.toString(36)` converts to
base-36 (alphanumeric). `.slice(-6)` takes the last 6 chars. Result: short,
lowercase, chronologically ordered, and safe even during template expansion
where multiple IDs are generated before any network write.

**Why 6 and not 4:** 4 chars = ~1.68M combinations. 6 chars = ~2.18B
combinations. During template application, 8+ IDs can be generated in the
same execution path before a GitHub write happens. At that point, GitHub 409
is too late — duplicate IDs may already be wired into dependency fields in
memory. 6 chars eliminates this risk while keeping filenames readable.

File name: `task-{id}-{slug}.md`
Example: `task-lx4k9m-seo-audit-homepage.md`

If a collision still occurs, the GitHub API returns 409. The extension retries
with a new timestamp. Zero extra logic required.

---

## 8. manifest.json — The Task Index

**Why it exists:** Reading 200+ task files via GitHub API to render the board
= 200+ API calls. Slow and expensive. One manifest read = one API call, full
board loaded instantly.

**What it contains:** Lightweight summary of every task — only the fields
needed for listing, filtering, and the board view. Full task body is only
fetched when a user opens a specific task.

```json
{
  "version": 1,
  "updated_at": "2026-03-11T14:30:00Z",
  "tasks": [
    {
      "id": "lx4k",
      "title": "SEO Audit - Homepage",
      "status": "in-progress",
      "priority": "high",
      "assignee": "@sarah",
      "created_by": "@iman",
      "due_date": "2026-03-17",
      "client": "acme",
      "project": "project-acme-rebrand",
      "labels": ["seo", "homepage"],
      "dependencies": ["n7q3"],
      "repeat": "none",
      "conflict_flag": false,
      "file": "task-lx4k-seo-audit-homepage.md"
    }
  ]
}
```

**Rules:**
- Manifest is a cache. Individual task files are always authoritative.
- JS updates manifest after every write (incremental — update one entry).
- If manifest gets out of sync, a `rebuildManifest()` function reads all
  task files and regenerates it from scratch. One click from the extension.
- AI can read manifest for quick queries without opening every file.
- Dashboard reads manifest. Only fetches full file when task is opened.

---

## 9. The Simple JS Functions (~130 lines total)

These live inside the Chrome extension. They are not a module, not a
package, not a separate project. They are functions.

```javascript
// 1. Generate a task ID
function generateId() {
  return Date.now().toString(36).slice(-6)
}

// 2. Recurring task trigger — the most important logic
// Called by the extension when a task is marked done
async function onTaskCompleted(task) {
  if (task.repeat === 'none' || !task.repeat) return

  const intervals = {
    daily: 1, weekly: 7, 'bi-weekly': 14, monthly: 30, quarterly: 90
  }
  const days = intervals[task.repeat]
  const newDueDate = addDays(task.due_date, days)

  await createTask({
    title: task.title,
    status: 'todo',
    priority: task.priority,
    assignee: task.assignee,
    client: task.client,
    project: task.project,
    labels: task.labels,
    repeat: task.repeat,
    due_date: newDueDate,
    spawned_from: task.id,
    checklist: resetChecklist(task.checklist)
  })
}

// 3. Manifest sync — called after every task write
async function updateManifest(updatedTask) {
  const manifest = await readManifest()
  const idx = manifest.tasks.findIndex(t => t.id === updatedTask.id)
  if (idx >= 0) {
    manifest.tasks[idx] = toManifestEntry(updatedTask)
  } else {
    manifest.tasks.push(toManifestEntry(updatedTask))
  }
  manifest.updated_at = new Date().toISOString()
  await writeManifest(manifest)
}

// 4. Conflict detection — called on every GitHub API write response
function isConflict(response) {
  return response.status === 409
}

// 5. Rebuild manifest from scratch
// Runs automatically on startup and after failures. Manual trigger also available.
async function rebuildManifest() {
  const files = await listTaskFiles()
  const tasks = await Promise.all(files.map(readAndParseTask))
  await writeManifest({ version: 1, updated_at: now(), tasks })
}

// 6. Save-time validator — runs before every GitHub write
// Catches the most common failure modes without becoming a full validation framework.
function validateTask(task, team) {
  const validStatus = ['todo','in-progress','waiting','in-review','done','archived']
  const validPriority = ['urgent','high','medium','low']
  const validRepeat = ['none','daily','weekly','bi-weekly','monthly','quarterly']
  const teamHandles = team.members.map(m => m.handle)

  const errors = []
  if (!task.title) errors.push('title is required')
  if (!validStatus.includes(task.status)) errors.push(`invalid status: ${task.status}`)
  if (!validPriority.includes(task.priority)) errors.push(`invalid priority: ${task.priority}`)
  if (task.repeat && !validRepeat.includes(task.repeat)) errors.push(`invalid repeat: ${task.repeat}`)
  if (task.assignee && !teamHandles.includes(task.assignee)) errors.push(`assignee ${task.assignee} not in TEAM.md`)

  return errors // empty array = valid. Show errors to user before write.
}
```

**Auto-recovery rules for `rebuildManifest()`:**
- Runs on extension startup (lightweight consistency check)
- Runs after offline queue replay completes
- Runs after any detected manifest write failure

Manual rebuild button still exists in extension settings as a fallback.

Everything else — understanding intent, applying templates, resolving
conflicts, answering questions — is handled by AI guided by SCHEMA.md.

---

## 10. Task File Format

YAML frontmatter + Markdown body. We keep YAML because:
- Industry standard for markdown metadata
- Renders as formatted table in GitHub's file viewer
- Reliable to parse with `gray-matter` (mature library, handles special chars)
- With manifest.json, we only parse individual files when needed — not 200 at once

```markdown
---
id: lx4k
title: SEO Audit - Homepage
status: in-progress
priority: high
assignee: "@sarah"
created_by: "@iman"
created_date: "2026-03-10"
updated_date: "2026-03-11"
due_date: "2026-03-17"
estimated_hours: 8
actual_hours: 3
client: acme
project: project-acme-rebrand
labels:
  - seo
  - homepage
dependencies:
  - n7q3
repeat: none
spawned_from: null
conflict_flag: false
attachments:
  - attachments/task-lx4k/current-rankings.pdf
---

## Context
Homepage needs a full SEO audit before the rebrand launch.

## Checklist
- [x] Keyword research
- [x] Competitor analysis
- [ ] On-page audit
- [ ] Page speed analysis
- [ ] Backlink audit
- [ ] Final report

## Acceptance Criteria
- All 6 checklist items completed
- Report delivered to client contact

## Time Log
| Date | Hours | By | Notes |
|---|---|---|---|
| 2026-03-10 | 2h | @sarah | Keyword research |
| 2026-03-11 | 1h | @sarah | Competitor analysis |

## AI Notes
> Written and maintained by AI only. Humans do not edit this section.

Acme's main gap is long-tail keyword coverage. Page speed at 67/100 —
likely a ranking factor. Recommend fixing speed before new content.

_@claude — 2026-03-11_
```

---

## 11. All Task Fields

| Field | Type | Required | Set by | Description |
|---|---|---|---|---|
| `id` | string | Yes | JS | Timestamp-based. 6 chars. e.g. `lx4k9m` |
| `title` | string | Yes | user/AI | Human-readable title |
| `status` | enum | Yes | user/AI | todo / in-progress / waiting / in-review / done / archived |
| `priority` | enum | Yes | user/AI | urgent / high / medium / low |
| `assignee` | string | No | user/AI | @username from TEAM.md |
| `created_by` | string | Yes | JS | @username who created |
| `created_date` | date | Yes | JS | ISO. Set once, never changed |
| `updated_date` | date | Yes | JS | ISO. Updated on every edit |
| `due_date` | date | No | user/AI | ISO deadline |
| `estimated_hours` | number | No | user/AI | Upfront estimate |
| `actual_hours` | number | No | JS | Summed from Time Log |
| `client` | string | No | user/AI | Client slug |
| `project` | string | No | user/AI | Matches projects/ file id |
| `labels` | array | No | user/AI | Free-form tags |
| `dependencies` | array | No | user/AI | Task IDs that must complete first |
| `repeat` | enum | No | user/AI | none / daily / weekly / bi-weekly / monthly / quarterly |
| `spawned_from` | string | No | JS | Parent recurring task ID |
| `conflict_flag` | boolean | Yes | JS | true if 409 detected. Default: false |
| `attachments` | array | No | user/AI | Paths in attachments/{id}/ |

---

## 12. TEAM.md Schema

```markdown
# Team

## Members

### @iman
- name: Iman
- role: Project Lead
- email: iman@agency.com
- capacity_hours_per_week: 40

### @sarah
- name: Sarah
- role: SEO Specialist
- email: sarah@agency.com
- capacity_hours_per_week: 35

## AI Identities
- @claude: Claude (Anthropic)
- @codex: Codex (OpenAI)
- @gemini: Gemini (Google)

## Roles
- Project Lead: create projects, assign tasks, modify templates
- Specialist: create and edit tasks, log time
```

---

## 13. Project File Schema

```markdown
---
id: project-acme-rebrand
title: Acme Rebrand
client: acme
status: active
start_date: "2026-03-01"
target_date: "2026-04-15"
lead: "@iman"
---

## Objective
Full brand refresh for Acme Corp.

## Key Contacts
- Client: Jane Smith (jane@acme.com)

## Milestones
- [x] Kickoff (March 1)
- [ ] Brand guidelines approved (March 15)
- [ ] Launch (April 15)

## Notes
Client prefers minimalist design. CEO approves personally — 3-5 day cycles.
```

Valid status: active / on-hold / completed / cancelled

---

## 14. Template Metadata (TEMPLATE.md)

```markdown
---
template_id: seo-audit
title: SEO Audit
category: seo
estimated_total_hours: 32
task_count: 8
default_duration_weeks: 3
---

## When to Use
New SEO client or periodic audit for existing client.

## Prerequisites
- Client website URL
- Google Search Console access
- Google Analytics access

## Task Order and Dependencies
1. keyword-research (8h) — no dependencies
2. competitor-analysis (6h) — no dependencies
3. on-page-audit (4h) — depends on 1
4. technical-audit (4h) — no dependencies
5. backlink-audit (4h) — depends on 1
6. content-gap-analysis (3h) — depends on 1, 2
7. page-speed (2h) — no dependencies
8. final-report (1h) — depends on all above

## AI Guidance
When applying this template:
- Ask for website URL and main competitors
- Ask which tasks to skip (not all clients need backlink audit)
- Calculate due dates backwards from the deadline
- Warn if deadline is less than 2 weeks (minimum is 14 days)
- Critical path is: keyword-research → on-page-audit → final-report (13 days)
```

---

## 15. SCHEMA.md — Full Content

```markdown
# Loop Schema v3

> Every AI must read this file before any operation in this repo.

## What Loop Is
Git-native project management for non-technical teams. Tasks are Markdown
files. A few JS functions handle deterministic rules. AI handles everything
that needs understanding. GitHub handles storage and history.

## Golden Rules
1. Read manifest.json for listing/filtering tasks. Read individual
   task files only when full detail is needed.
2. Never edit SCHEMA.md, TEAM.md, or config.yml unless explicitly asked.
3. One task per file. Never combine tasks in one file.
4. Use only field values defined in this schema.
5. The JS layer (in the extension/dashboard) handles: ID generation,
   recurring task creation, manifest updates, conflict flag. Do not
   replicate this logic — it runs automatically.
6. When creating tasks via direct file write (not through extension),
   generate an ID using: Date.now().toString(36).slice(-6)
   Then update manifest.json manually.

## Valid Status Values
- todo          → not started
- in-progress   → being worked on
- waiting       → blocked on someone/something
- in-review     → submitted for review
- done          → completed
- archived      → no longer relevant

## Valid Priority Values
- urgent   → drop everything
- high     → this week
- medium   → this sprint
- low      → someday

## Valid Repeat Values
- none, daily, weekly, bi-weekly, monthly, quarterly

## Conflict Resolution (AI handles this on request — best-effort in v1)
Conflict resolution in v1 is best-effort, not deterministic. Inform the user
of this when resolving conflicts.

1. Read both file versions from Git history
2. Prefer most recent non-null value per field
3. Merge checklists: include all items, most recent completion state
   (note: two similar items edited independently may not merge cleanly)
4. Merge Time Logs: append all entries, remove exact duplicates only
   (note: two entries by the same person on the same day are kept both)
5. Append AI Notes from both versions with timestamps
6. Write resolved version, set conflict_flag: false
7. Note resolution in AI Notes, flag any items requiring human review

**v2 improvement:** Add lightweight `entry_id` to time log rows and checklist
items to enable precise deduplication. Not in scope for v1.

## Recurring Tasks (JS handles this automatically)
When status is set to done and repeat != none, the extension/dashboard
automatically creates the next occurrence. AI does not need to do this —
it is handled by the JS layer. AI should confirm to the user that the
next task will be created.

## Template Behavior
1. Read the template folder in templates/
2. For each task file: replace PLACEHOLDER values with real data
3. Assign new IDs to each task (Date.now().toString(36).slice(-4))
4. Wire dependencies using new IDs
5. Set project, client, assignee on all tasks
6. Calculate due dates backwards from deadline using task order in TEMPLATE.md
7. Update manifest.json with all new tasks
8. Report to user: tasks created, critical path, buffer time available

## What AI Is Good At Here
- Understanding "mark the SEO thing as done" → resolves to task lx4k
- Answering "who is overloaded this week?" → reads manifest, sums hours
- "What's blocking the launch?" → traces dependency chain
- "Should I be worried about our timeline?" → reads dates, estimates, flags risk
- Applying templates with contextual customization
- Writing AI Notes with domain-specific insight
- Resolving conflicts with judgment

## What JS Handles (not AI)
- ID generation
- manifest.json updates after writes
- Recurring task creation trigger
- GitHub 409 conflict flag

## Attachment Rules
Files go in attachments/{task-id}/. Reference in frontmatter.
GitHub API max: 100MB per file. Base64 encoded — large files are slow.
Recommended: keep attachments under 5MB for performance.

## Team and Project References
- Usernames: always @username format, must exist in TEAM.md
- AI identities: @claude, @codex, @gemini
- Projects: must match id field in a projects/ file
```

---

## 16. GitHub Integration

**Auth:** GitHub OAuth via Chrome extension. `repo` scope. Token in
`chrome.storage.local`. Never sent anywhere except GitHub API.

**Key API calls:**

| Action | Call | Notes |
|---|---|---|
| Read task | GET /contents/tasks/{file} | Returns content + SHA |
| Write task | PUT /contents/tasks/{file} | Must include SHA for updates |
| Read manifest | GET /contents/tasks/manifest.json | One call, full state |
| Write manifest | PUT /contents/tasks/manifest.json | After every task change |
| List files | GET /contents/tasks/ | Only used for rebuildManifest() |
| Upload attachment | PUT /contents/attachments/{id}/{file} | Base64 encoded |
| Read history | GET /commits?path=tasks/{file} | For conflict resolution |

**Rate limits:** 5,000 req/hr per user. With manifest, most operations are
3 calls (read task, write task, update manifest). 10 people × 50 ops/hr
= 1,500 calls. Well within limits.

**Offline:** Extension caches manifest locally. Browse tasks (read-only)
while offline. Failed writes queue in `chrome.storage.local`, retry on
reconnect. "Pending sync" indicator shown.

---

## 17. Chrome Extension Architecture

**Tech stack:**
- Manifest V3
- TypeScript
- Preact for UI (lightweight, ~4KB)
- gray-matter for YAML parse/write
- GitHub REST API v3
- GitHub OAuth via `chrome.identity`

**Permissions:**
- `storage` — token, cached manifest, offline queue
- `sidePanel` — persistent side panel
- `contextMenus` — right-click task creation
- `activeTab` — page context for task creation
- `notifications` — conflict, overdue, recurring alerts
- `identity` — GitHub OAuth

**Core features:**
- Side panel: Kanban + list + by-assignee (reads manifest)
- Quick add: Cmd+Shift+L
- Right-click: "Add Loop task from this page" (attaches URL + title)
- AI chat: user message + manifest → AI provider → tool calls → JS functions
- Offline queue: failed writes stored and retried
- Notifications: conflict badge, overdue tasks, recurring reminders

---

## 18. Dashboard (dashboard.html)

Self-contained HTML + JS file. No build step. No server. One file.

Reads manifest.json from GitHub API (requires user's GitHub token on first
load, stored in localStorage). Renders board client-side from manifest data.
Only fetches individual task files when a task is opened.

> ⚠️ **Internal team use only (v1).** The GitHub token is stored in
> `localStorage`, which is weaker than the extension's `chrome.storage.local`
> model. This is acceptable for a private team dashboard accessed on trusted
> devices. Do not use this pattern for client-facing access or shared/public
> deployments. Client-facing dashboard (v2) will use a Cloudflare Worker proxy
> so no token is ever exposed to clients.

**v1 views:** Kanban (by status), List (sortable), By-assignee
**v2 views:** By-client, Timeline, Client-facing (via Cloudflare Worker proxy)

Read-only in v1. All writes through Chrome extension or AI.

---

## 19. config.yml

```yaml
project:
  name: "Acme Agency"
  timezone: "America/Los_Angeles"
  default_priority: medium

github:
  owner: "acme-agency"
  repo: "loop"
  branch: main

tasks:
  auto_archive_done_after_days: 30
  warn_overdue_after_hours: 24

definition_of_done:
  - Reviewed by assignee
  - Client notified if applicable
  - Attachments uploaded if relevant

notifications:
  conflict_alert: true
  overdue_alert: true
  recurring_reminder_hours_before: 24
```

---

## 20. Build Phases

### Phase 1 — Foundation (Weeks 1–3)
Working repo + working dashboard. Usable via direct AI conversation today.

- [ ] Write SCHEMA.md, AGENTS.md, TEAM.md, config.yml, project schema
- [ ] Write 2 starter templates: seo-audit, client-onboarding
- [ ] Build repo initializer (writes full structure via GitHub API)
- [ ] Build dashboard.html (reads manifest, Kanban view)
- [ ] Test with Claude: create, edit, complete, recurring tasks
- [ ] Test with Codex: same operations, verify manifest stays consistent
- [ ] Document gaps found during testing

**Success:** Non-technical person connects repo, asks Claude to set up a
project, manages it entirely through conversation. Dashboard shows correct state.

### Phase 2 — Chrome Extension (Weeks 4–7)
Native app feel, no installation.

- [ ] Extension scaffold (Manifest V3, TypeScript)
- [ ] GitHub OAuth
- [ ] The 5 JS functions (generateId, onTaskCompleted, updateManifest,
      isConflict, rebuildManifest)
- [ ] Side panel: Kanban from manifest
- [ ] Task detail: load full file on open
- [ ] Create/edit task forms
- [ ] Quick add (Cmd+Shift+L)
- [ ] Right-click page task creation
- [ ] AI chat panel (Claude API, tool calls)
- [ ] Offline queue
- [ ] Conflict + overdue notifications
- [ ] Real user test: 5-person agency team, 2 weeks, real client project

**Success:** Zero Git commands, zero terminal usage by any team member.

### Phase 3 — Templates + Intelligence (Weeks 8–11)

- [ ] All 6 templates with AI guidance in TEMPLATE.md
- [ ] Template application via AI conversation
- [ ] Proactive suggestions (stale tasks, overload warnings, blocked deps)
- [ ] Historical intelligence ("SEO audits take us avg 28h")
- [ ] Timeline view in dashboard
- [ ] By-client view

### Phase 4 — Scale + Launch (Weeks 12–16)

- [ ] Multi-repo (one extension, multiple client repos)
- [ ] Client dashboard via Cloudflare Worker (safe token proxy)
- [ ] Template marketplace
- [ ] 500+ task performance test
- [ ] Documentation site + landing page
- [ ] Public beta

---

## 21. Response to Codex Technical Review (v1 → v2 → v3)

This section documents what Codex raised, what we agreed with, and where
we pushed back. Other AI reviewers: please add your notes in Section 22.

**"AI replaces the backend is the weakest assumption"**

Partially agree. We agree that a few specific things need JS, not AI:
ID generation, recurring task trigger, manifest sync, 409 detection.
We disagree that we need a full deterministic "loop-core" module.
That was over-engineering. ~100 lines of JS inside the extension is enough.
AI handles validation, template logic, conflict resolution — guided by SCHEMA.md.
A wrong priority label is not a bank transaction. SCHEMA.md is clear enough.

**"Auto-increment IDs will collide"**

Agree. Fixed. Using timestamp-based IDs (Date.now().toString(36).slice(-4)).
Virtually collision-free. GitHub 409 handles the edge case automatically.

We chose this over ULIDs because ULIDs are 26 chars and ugly for non-technical
users browsing files. Short IDs (lx4k, m2p9) are readable and lightweight.

**"Search/filter performance is a real risk"**

Agree. Fixed. manifest.json solves this. One API call loads the full board.
Individual files only fetched when a task is opened.

**"Auth/security — embedded token is dangerous"**

Agree. Removed. v1 is internal team only. Client-facing dashboard is v2,
built via Cloudflare Worker proxy. Token never exposed to clients.

**"AI execution model is underspecified"**

Agree. Now specified in detail: Claude API (Anthropic), user brings own key,
stored in chrome.storage.local, never hits a Loop server (there is no Loop
server), tool calls are specific GitHub operations routed through JS functions,
AI failure falls back to read-only cached manifest mode.

**"Missing schemas for projects/ and TEAM.md"**

Agree. Now fully specified in Sections 12, 13, 14.

**What Codex suggested that we pushed back on:**

A full loop-core module with validateTask(), parseTaskFile(), writeTaskFile(),
detectConflict() as a separate deterministic layer. This is rebuilding
Backlog.md's CLI inside the extension. The whole point of Loop is that AI
handles the logic. The Chrome extension is not an npm package. We keep it
simple: 5 targeted functions, ~100 lines, not a module.

---

## 22. AI Reviewer Notes

> This section is for AI reviewers to add signed notes, questions, and
> suggestions. Add your notes below with your identifier and date.
> Humans: paste AI feedback here or have the AI edit this file directly
> via GitHub once the repo is live.

---

### @claude — 2026-03-10
Authored v1, v2, v3. Key decisions I'm least certain about:

1. **gray-matter as the YAML parser** — it's the right choice technically,
   but it adds a dependency to the extension. Worth confirming it handles
   all edge cases users will hit (task titles with colons, hashtags, quotes).

2. **4-char timestamp ID** — Date.now().toString(36) produces 8-9 chars.
   .slice(-4) takes the last 4. In base-36, 4 chars = 1.68M combinations.
   At 1000ms resolution, two tasks created in the same millisecond would
   collide. GitHub 409 handles this. But worth stress-testing.

3. **manifest.json as cache vs. source of truth** — I called it a cache
   (files are authoritative). This means if manifest gets out of sync,
   `rebuildManifest()` is the recovery. The risk is: what if the extension
   crashes mid-write (task written, manifest not updated)? rebuildManifest()
   should run on every extension startup as a consistency check.
   Small, but worth noting.

4. **SCHEMA.md length** — as Loop evolves and schema grows, SCHEMA.md will
   get long. AI context windows are large now but not infinite. Worth
   watching and potentially splitting into SCHEMA.md (rules) + FIELDS.md
   (field reference) at some point.

---

### @codex — 2026-03-10
v3 is materially better than v1. The architecture now has a credible split:
GitHub stores state, JS handles a small deterministic surface area, and AI
handles interpretation and workflow. That is the right direction.

My remaining notes are narrow and practical:

1. **Task ID generation still needs one more safety margin.**
   `Date.now().toString(36).slice(-4)` is fine for single task creation, but
   template expansion can generate several IDs in one execution path before a
   network write happens. In that case, GitHub 409 is too late because the
   duplicate IDs may already be wired into dependencies in memory.

   **Recommended fix:** keep the same idea, but make IDs slightly safer:
   - Option A: use 6 chars instead of 4
   - Option B: use 4 chars + a per-session counter when multiple tasks are
     created in the same millisecond

   This keeps IDs human-readable without introducing ULID-length filenames.

2. **manifest.json recovery should be automatic, not manual.**
   Calling manifest a cache is correct. But if a task write succeeds and
   manifest write fails, the board becomes stale until a human notices.

   **Recommended fix:** run `rebuildManifest()` automatically:
   - on extension startup
   - after offline queue replay
   - after any detected manifest write failure

   Manual rebuild can still exist, but it should not be the main recovery path.

3. **Conflict merge rules need stable item identity if you want reliable merges.**
   "Deduplicate by date+by" for time logs is not strong enough. Two entries by
   the same person on the same day can be different work. Checklist merging has
   the same problem if two similar items are edited independently.

   **Recommended fix:** if conflict resolution remains a real feature in v1,
   add lightweight IDs:
   - time log rows get a small `entry_id`
   - checklist items get a small internal ID or structured representation

   If that feels too heavy, then reduce the promise: document conflict
   resolution as "best-effort for v1" rather than deterministic.

4. **A minimal save-time validator is still worth having.**
   I am not arguing for a separate `loop-core` package. I agree that would be
   too much for this product. But a few checks before write will prevent the
   easiest failure modes:
   - invalid YAML frontmatter
   - invalid `status`, `priority`, or `repeat`
   - missing required fields
   - `assignee` not found in TEAM.md

   This can stay small and local to the extension. It does not need to become a
   framework or module.

5. **The dashboard token model is acceptable for internal v1, but should be
   labeled explicitly as internal-only.**
   Storing a GitHub token in `localStorage` is weaker than the extension model.
   That is probably acceptable for a private team dashboard in v1, but the doc
   should say so plainly to avoid this pattern leaking into anything broader.

Summary judgment: I think this is now buildable as a v1. The remaining work is
not conceptual. It is mostly about tightening a few reliability edges so the
product behaves consistently when real teams use it.

---

### @replit — [add your notes here]

Context for Replit: this is a GitHub-native task management system for
non-technical teams. No backend server. The Chrome extension and a standalone
dashboard.html are the two interfaces. The extension uses ~130 lines of plain
JS for deterministic logic. AI (any model) handles the rest, guided by
SCHEMA.md. Please review with that framing — your input on the JS functions,
the manifest pattern, and the dashboard token model would be most valuable.

---

### [other reviewers — add here]

---

## 23. What Loop Is NOT

- Not a replacement for Google Docs (no real-time co-editing)
- Not a replacement for Slack (no messaging)
- Not a replacement for Jira (not for enterprise or dev teams)
- Not a SaaS (no server, no subscription, no Loop infrastructure)
- Not a CLI tool (no terminal ever)
- Not Backlog.md (they serve developers; we serve agencies)

---

## 24. Lessons from Backlog.md

We studied Backlog.md before building Loop.

**Borrowed:**
- One file per task in a dedicated folder — the core structural insight
- decisions/ folder for recording why, not just what
- AGENTS.md progressive disclosure — read schema first, then act
- Definition of Done as project-wide config

**Avoided:**
- CLI as primary interface (requires install)
- Auto-increment IDs (collision-prone)
- No index/manifest (slow at scale)
- Naive YAML parsing (special char bugs — use gray-matter)
- Developer-only scope (no client fields, no time tracking, no templates)
- Web UI as afterthought (broken drag-and-drop, broken deep links)

**Added that they haven't:**
- manifest.json for fast reads
- Timestamp-based IDs
- Recurring tasks (JS trigger)
- Template metadata with AI guidance
- Chrome extension as primary interface
- SCHEMA.md as AI contract
- Project and team schemas
- conflict_flag visible on dashboard
- AI Notes section per task
- Time tracking (estimated vs actual hours)
- Client and project hierarchy

---

*This document is the single source of truth for Loop's architecture.*
*Share with any AI or engineer for technical review.*
*AI reviewers: please add signed notes in Section 22.*

*Last updated: 2026-03-10 by @iman (via @claude — v3.1)*
