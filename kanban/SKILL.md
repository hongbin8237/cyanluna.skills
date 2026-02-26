---
name: kanban
description: Manage project tasks in per-project kanban DBs (~/.claude/kanban-dbs/{project}.db). Supports task CRUD (add, edit, move, remove), board viewing, session context persistence, and statistics. For pipeline orchestration use /kanban-run, for requirements refinement use /kanban-refine. Run /kanban-init first to register the project.
license: MIT
---

> Shared context: read `~/.claude/skills/kanban/shared.md` for DB path, pipeline levels, status transitions, API endpoints, error handling, and agent context flow.

## Commands

### `/kanban` or `/kanban list` — View Board

```bash
BOARD=$(curl -s "http://localhost:5173/api/board?project=$PROJECT")
```

Fallback (no dev server):
```bash
sqlite3 -header -column ~/.claude/kanban-dbs/$PROJECT.db \
  "SELECT id, title, status, priority FROM tasks WHERE project='$PROJECT' \
   ORDER BY CASE status WHEN 'impl' THEN 0 WHEN 'impl_review' THEN 1 \
   WHEN 'plan' THEN 2 WHEN 'plan_review' THEN 3 WHEN 'test' THEN 4 \
   WHEN 'todo' THEN 5 WHEN 'done' THEN 6 END, id"
```

Output: markdown table with ID, Status, Priority, Title.

### `/kanban context` — Session Handoff

**Run first when starting a new session.** Fetch board and output pipeline state:
Implementing / Plan Review / Impl Review / Testing / Recently Done / Next Todo.

### `/kanban add <title>` — Add Task

1. Ask user for priority, level (L1/L2/L3), description, tags (use AskUserQuestion)
2. POST to API, output confirmation with new task ID

### `/kanban move <ID> <status>` — Move Task

```bash
curl -s -X PATCH "http://localhost:5173/api/task/$ID?project=$PROJECT" \
  -H 'Content-Type: application/json' \
  -d "{\"status\": \"$STATUS\"}"
```

The API enforces valid transitions. Invalid moves return 400 with allowed transitions.

### `/kanban edit <ID>` — Edit Task

Ask user which fields to modify, then PATCH via API.

### `/kanban remove <ID>` — Delete Task

```bash
# API (preferred)
curl -s -X DELETE "http://localhost:5173/api/task/$ID?project=$PROJECT"

# sqlite3 fallback
sqlite3 ~/.claude/kanban-dbs/$PROJECT.db "DELETE FROM tasks WHERE id=$ID;"
```

### `/kanban stats` — Statistics

```bash
BOARD=$(curl -s "http://localhost:5173/api/board?project=$PROJECT")
echo "$BOARD" | jq '{
  todo: (.todo | length), plan: (.plan | length),
  plan_review: (.plan_review | length), impl: (.impl | length),
  impl_review: (.impl_review | length), test: (.test | length),
  done: (.done | length),
  total: ((.todo + .plan + .plan_review + .impl + .impl_review + .test + .done) | length)
}'
```

## Setup & Web Board

Run `/kanban-init` first to register this project.

Add to `.gitignore`:
```
.claude/kanban.json
kanban-board/
```

Start web board: `./kanban-board/start.sh` → `http://localhost:5173/?project=<PROJECT>`
Features: 7-column pipeline, drag-and-drop (valid transitions only), card lifecycle modal, agent log viewer, 10s auto-refresh.
