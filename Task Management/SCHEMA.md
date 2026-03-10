# Loop Schema v3.1

> **Every AI must read this file before any operation in this repo.**
> This is the contract. Follow it exactly. Do not guess field names or values.

---

## What Loop Is

Git-native project management for non-technical teams. Tasks are Markdown files
stored in GitHub. A few JS functions handle deterministic rules (ID generation,
manifest sync, recurring tasks, conflict detection). AI handles everything that
needs understanding or judgment. GitHub handles storage, history, and access.

**The stack:**
- `tasks/*.md` — one file per task
- `tasks/manifest.json` — lightweight index for fast reads
- `TEAM.md` — team roster and roles
- `config.yml` — project settings
- `templates/` — repeatable workflow templates with AI guidance
- `projects/` — project files linking tasks together
- `dashboard.html` — self-contained read-only board (no build step)

---

## Golden Rules

1. Read `tasks/manifest.json` for listing, filtering, and board views.
   Read individual task files only when full detail is needed.
2. Never edit `SCHEMA.md`, `TEAM.md`, or `config.yml` unless explicitly asked.
3. One task per file. Never combine tasks in one file.
4. Use only field values defined in this schema. No freeform status values.
5. The JS layer (Chrome extension / dashboard) handles automatically:
   ID generation, manifest updates, recurring task creation, conflict flag.
   Do not replicate this logic — it runs automatically.
6. When creating tasks via direct file write (not through the extension),
   generate an ID using: `Date.now().toString(36).slice(-6)`
   Then update `tasks/manifest.json` manually.
7. Always sign AI Notes with `@handle — YYYY-MM-DD`.

---

## Valid Field Values

### status
| Value | Meaning |
|---|---|
| `todo` | Not started |
| `in-progress` | Being actively worked on |
| `waiting` | Blocked on someone or something external |
| `in-review` | Submitted for review or approval |
| `done` | Completed |
| `archived` | No longer relevant, kept for history |

### priority
| Value | Meaning |
|---|---|
| `urgent` | Drop everything |
| `high` | This week |
| `medium` | This sprint / this month |
| `low` | Someday / backlog |

### repeat
`none` · `daily` · `weekly` · `bi-weekly` · `monthly` · `quarterly`

---

## Task File Format

```
tasks/task-{id}-{slug}.md
```

YAML frontmatter followed by Markdown body sections.

### Required frontmatter fields
- `id` — 6-char base-36 timestamp string (e.g. `lx4k9m`)
- `title` — human-readable task title
- `status` — see valid values above
- `priority` — see valid values above
- `created_by` — @handle of creator
- `created_date` — ISO date string
- `updated_date` — ISO date string, update on every edit
- `conflict_flag` — boolean, default `false`

### Optional frontmatter fields
- `assignee` — @handle from TEAM.md
- `due_date` — ISO date string
- `estimated_hours` — number
- `actual_hours` — number (summed from Time Log)
- `client` — client slug
- `project` — matches `id` field in a `projects/` file
- `labels` — array of free-form strings
- `dependencies` — array of task IDs that must complete first
- `repeat` — see valid values above
- `spawned_from` — task ID of parent recurring task
- `attachments` — array of paths under `attachments/{task-id}/`

### Body sections (Markdown)

```markdown
## Context
Why this task exists. Background the assignee needs.

## Checklist
- [ ] Item one
- [ ] Item two

## Acceptance Criteria
What done looks like.

## Time Log
| Date | Hours | By | Notes |
|---|---|---|---|
| YYYY-MM-DD | 2h | @handle | What was done |

## AI Notes
> Written and maintained by AI only. Humans do not edit this section.

Insight, observations, flags.

_@handle — YYYY-MM-DD_
```

---

## Conflict Resolution (best-effort in v1)

When `conflict_flag: true` appears on a task, AI resolves it on request.
Inform the user that v1 resolution is best-effort, not deterministic.

1. Read both file versions from Git history
2. Prefer most recent non-null value per field
3. Merge checklists: include all items, most recent completion state
   (note: independently edited similar items may not merge cleanly)
4. Merge Time Logs: append all entries, remove exact duplicates only
   (note: two entries by same person on same day are kept as separate rows)
5. Append AI Notes from both versions with timestamps
6. Write resolved version, set `conflict_flag: false`
7. Note resolution in AI Notes, flag anything requiring human review

---

## Recurring Tasks

When a task has `repeat != none` and its status is set to `done`, the Chrome
extension automatically creates the next occurrence. AI does not need to trigger
this — it happens in JS. When marking a recurring task done, AI should confirm
to the user: "The next occurrence will be created automatically."

---

## Template Behavior

When applying a template from `templates/`:

1. Read `templates/{name}/TEMPLATE.md` for metadata and AI guidance
2. For each task file in the template folder: copy and replace PLACEHOLDER values
3. Generate new IDs: `Date.now().toString(36).slice(-6)` — one per task
4. Wire dependencies using the new IDs
5. Set `project`, `client`, `assignee` on all tasks
6. Calculate due dates backwards from the deadline using task order in TEMPLATE.md
7. Update `tasks/manifest.json` with all new task entries
8. Report to user: tasks created, critical path, buffer time available

---

## What AI Is Responsible For

- Understanding natural language references ("the SEO thing", "the Acme project")
- Resolving @mentions to team members from TEAM.md
- Applying templates with contextual customization
- Answering workload, timeline, and dependency questions
- Writing AI Notes with domain-specific insight
- Resolving conflicts with judgment
- Flagging risks proactively (overdue, blocked, overloaded)

## What JS Handles (not AI)

- ID generation (`Date.now().toString(36).slice(-6)`)
- `manifest.json` updates after every write
- Recurring task creation trigger
- GitHub 409 conflict flag (`conflict_flag: true`)
- Save-time validation (status/priority/repeat enums, required fields, team check)
- Manifest auto-rebuild on startup and after failures

---

## Team and Project References

- Usernames: always `@handle` format — must exist in TEAM.md
- AI identities: `@claude`, `@codex`, `@gemini`, `@replit`
- Projects: `project` field must match `id` in a `projects/` file
- Clients: `client` field is a slug (lowercase, hyphenated)

---

## Attachment Rules

Files go in `attachments/{task-id}/`. Reference paths in the `attachments`
frontmatter field. GitHub API max: 100MB per file (base64 encoded). Keep
attachments under 5MB for acceptable performance.

---

## manifest.json Structure

```json
{
  "version": 1,
  "updated_at": "YYYY-MM-DDTHH:MM:SSZ",
  "tasks": [
    {
      "id": "lx4k9m",
      "title": "Task title",
      "status": "todo",
      "priority": "high",
      "assignee": "@handle",
      "created_by": "@handle",
      "due_date": "YYYY-MM-DD",
      "client": "client-slug",
      "project": "project-id",
      "labels": [],
      "dependencies": [],
      "repeat": "none",
      "conflict_flag": false,
      "file": "task-lx4k9m-task-slug.md"
    }
  ]
}
```

Manifest is a cache. Individual task files are always authoritative.
If manifest is out of sync, use `rebuildManifest()` in the extension.

---

_Schema version: 3.1 — Last updated: 2026-03-10_
_This file is the AI contract for Loop. All AIs operating in this repo must follow it._
