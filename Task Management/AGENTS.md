# Loop — AI Agent Orientation

> You are operating in a Loop workspace.
> Read this file first. Then read SCHEMA.md. Then act.

---

## What This Repo Is

This is a Loop project management workspace for **Lemniscate Marketing**.
Tasks, projects, and team data are stored as Markdown files in GitHub.
You interact with this workspace through conversation.

## Step 1 — Always Read SCHEMA.md First

Before creating, editing, querying, or reporting on anything:

```
Read SCHEMA.md
```

SCHEMA.md defines all valid field values, file formats, task structure,
manifest behavior, and rules for AI behavior. Do not operate without it.

## Step 2 — Use manifest.json for Queries

For any request about tasks (listing, filtering, status checks, workload,
overdue, blocked):

```
Read tasks/manifest.json
```

This is one API call and gives you the full board state. Only read individual
task files when you need the full body (context, checklist, time log, AI notes).

## Step 3 — Read TEAM.md Before Assigning

Before assigning tasks or answering workload questions:

```
Read TEAM.md
```

All @handles, roles, and capacity data are here. Never assign to a handle
that doesn't exist in TEAM.md.

---

## What You Can Do

| Request | What to do |
|---|---|
| "Create a task for..." | Read SCHEMA.md → write task file → update manifest |
| "What's the status of..." | Read manifest.json → answer |
| "Who is overloaded?" | Read manifest.json + TEAM.md → sum hours → flag |
| "Mark X as done" | Read task file → update status → update manifest → confirm recurring if applicable |
| "Apply the SEO audit template" | Read templates/seo-audit/TEMPLATE.md → create all tasks → update manifest |
| "What's blocking the launch?" | Read manifest.json → trace dependency chain → report |
| "Show me Acme tasks" | Read manifest.json → filter by client: acme → list |
| "Add a time log entry" | Read task file → append to Time Log → update actual_hours → write → update manifest |
| "Resolve the conflict on task X" | Read both versions from Git → merge per SCHEMA.md rules → write → confirm |

## What You Must Not Do

- Do not edit `SCHEMA.md`, `TEAM.md`, or `config.yml` unless explicitly asked
- Do not combine multiple tasks in one file
- Do not use field values not defined in SCHEMA.md
- Do not replicate JS logic (ID generation, manifest updates, recurring triggers)
  — these run automatically in the extension
- Do not assign tasks to handles not in TEAM.md

---

## AI Identity in This Repo

Sign all AI Notes with your handle and today's date:

```
_@claude — 2026-03-10_
_@codex — 2026-03-10_
_@replit — 2026-03-10_
```

Only write in the `## AI Notes` section of task files. Never edit human-written
sections (Context, Checklist, Acceptance Criteria, Time Log) unless asked.

---

## If You Are Unsure

1. Check SCHEMA.md for the rule
2. If the rule isn't there, ask the human before acting
3. If you make a change you're unsure about, note it in AI Notes with your reasoning

---

_This file is for AI agents. Humans: see LOOP-ARCHITECTURE.md for full design documentation._
