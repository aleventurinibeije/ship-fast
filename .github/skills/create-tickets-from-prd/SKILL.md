---
name: create-tickets-from-prd
description: 'Scan PRD.md for BE development activities that do not yet have a ticket reference, create a Redmine ticket for each one (with module tickets as parents), and update PRD.md with the ticket numbers. Use when asked to "create tickets from PRD", "sync PRD with tracker", "ticket PRD items", "generate tickets".'
---

# Create Tickets from PRD

## When to Use

Use this skill to:
1. Scan `.github/context/PRD.md` for **module activity rows** that have **no ticket reference** (`#\d+`)
2. Create a Redmine **Feature ticket per module** (M1, M2, …) as the parent
3. Create a Redmine **Task ticket per activity row** (M1-01, M2-03, …) as a child of the module ticket
4. Update `PRD.md` in-place to embed the ticket reference in each activity row

**Input:** Optional scope hint (e.g., "only M3", "only M6 and M7")
**Output:** Updated `.github/context/PRD.md` with `#[id]` references; created tickets on Redmine

---

## PRD Structure

The PRD uses this structure — **not** User-Story headings:

```markdown
### M1 — Infrastructure & Foundation

> Module description

| ID | Activity | Notes |
|----|----------|-------|
| M1-01 | Short description | Extra notes |
| M1-02 | Short description | Extra notes |
```

**A ticket must be created for every activity row** (`M{n}-{nn}`) that does not yet contain a `#\d+` reference anywhere in its row.

A row that already has a reference looks like:

```markdown
| M1-01 `#123` | Short description | Extra notes |
```

---

## Prerequisites

Read before starting:
- `.github/context/PRD.md` — authoritative source of development work
- `.github/context/providers/ticket-manager.md` — Redmine API, constants, output format rules
- `.github/context/architecture.md` — tech stack, module layout, file paths, auth layers, RBAC rules

---

## Steps

### Step 0 — Read Provider Config

Follow the **Standard Provider Loading Steps** from `.github/context/providers/LOADING-PROTOCOL.md` for the **ticket-manager** provider (`.github/context/providers/ticket-manager.md`).

This skill uses REST (curl), not MCP — skip the `tool_search_tool_regex` call. Read the provider config only to extract:
- `Base URL` → `REDMINE_BASE_URL`
- `API key env var` → `REDMINE_API_KEY`
- `Project identifier` → `PROJECT_ID`
- `Ticket pattern` → regex for detecting existing references (e.g. `#\d+`)
- Tracker IDs and status IDs → from the provider config's `## Constants` section
- `create-ticket` operation parameters (endpoint, body fields, response fields)

> **Security:** Never hardcode the API key. Use it as a shell variable loaded from `.env`.

---

### Step 1 — Read and Parse PRD

1. Read `.github/context/PRD.md` fully.

2. Build a **module inventory**: for each `### M{n} — <title>` heading, extract:
   - Module ID (e.g. `M1`)
   - Module title (e.g. `Infrastructure & Foundation`)
   - Module description (the `>` blockquote immediately below the heading)
   - Whether the **module heading line** already contains `#\d+` → if yes, the parent ticket ID is already known; extract it

3. For each module, scan its activity table rows:
   ```
   | M1-01 | Short description | Extra notes |
   ```
   Extract:
   - Activity ID (e.g. `M1-01`)
   - Activity description (second column)
   - Activity notes (third column)
   - Whether a `#\d+` reference already appears in the row

4. Build two maps:
   - **modules WITHOUT parent ticket** → need a parent ticket created
   - **activities WITHOUT ticket reference** → need a child ticket created

5. If the user specified a scope (`"only M3"`, `"M6 and M7"`), filter both maps to those modules only.

6. If **everything already has references**, report this and stop.

---

### Step 2 — Confirm Scope with User (if needed)

If no scope was specified and the pending list is long (> 20 activities), show a summary and ask:

> "I found N modules with M activities without tickets. I'll create a parent ticket per module and a child task per activity. Proceed with all, or would you like to limit to specific modules?"

Proceed immediately if the intent is clear or scope is small.

---

### Step 3 — Prepare Ticket Data

#### 3a — Module (parent) tickets

For each module **without** an existing parent ticket:

| Field | Value |
|-------|-------|
| `subject` | `[M{n}] {Module title}` — e.g. `[M1] Infrastructure & Foundation` |
| `description` | Markdown block — see template below |
| `tracker_id` | from provider Constants (`tracker_id_epica` or equivalent) |
| `status_id` | from provider Constants (`status_id_open` or equivalent) |
| `project_id` | from provider Constants (`project_id`) |

**Description template (Markdown) for module tickets:**

```markdown
## Description

{Module blockquote description from PRD}

## Tech Stack

{Copy the Tech Stack table from `.github/context/CONTEXT-SNAPSHOT.md` → Tech Stack section}

## Activities

| ID | Activity | Notes |
|--|--|--|
| M{n}-01 | {Activity description} | {Notes} |
| M{n}-02 | {Activity description} | {Notes} |
```

Include all activity rows for the module in the table.

#### 3b — Activity (child) tickets

For each activity **without** an existing ticket reference:

| Field | Value |
|-------|-------|
| `subject` | `[M{n}-{nn}] {Activity description}` — e.g. `[M1-01] Audit and complete existing SQL migrations (001–004)` |
| `description` | Markdown block with: activity notes, module context, any relevant PRD table rows (DB fields, endpoint path, auth scope) |
| `tracker_id` | from provider Constants (`tracker_id_task` or equivalent) |
| `status_id` | from provider Constants (`status_id_open` or equivalent) |
| `project_id` | from provider Constants (`project_id`) |
| `parent_issue_id` | ID of the module parent ticket created in Step 4a |

**Description template (Markdown) for activity tickets:**

```markdown
## Context

Module: **[M{n}] {Module title}**
Activity: **{Activity ID}** — {Activity description}

## Notes

{Activity notes from PRD, verbatim}

## Development Scope

### Stack & Conventions

{Copy the Mandatory Conventions section from `.github/context/CONTEXT-SNAPSHOT.md`. Include key file locations relevant to this activity type.}

### Implementation Notes

{Activity-type specifics — expand one of the blocks below, delete the others}

**If this is an endpoint:**

- HTTP method + path: `{METHOD} /admin/{path}`
- Required scope: `{resource}:{namespace}:{action}` — enforced with `fastify.requireScope('{scope}')`
- Auth layer: calls `fastify.requireBuildingAccess()` if tenant-scoped
- Request schema: `{RequestSchema}` (TypeBox in `schemas/`)
- Response schema: `{ResponseSchema}`

**If this is a migration:**

- Migration file: `{migrations-path}/{NNN}.do.sql` (path from project conventions)
- Table(s) affected: `{table_name}`
- New columns / constraints: {list changes}
- Undo migration: `{NNN}.undo.sql` must revert all changes cleanly

**If this is a test task:**

- Test file: `{test-path}/{domain}.test.{ts|js}` (path from project conventions)
- Uses project test bootstrap helper for isolated setup
- Scenarios to cover: {list test cases from PRD notes}
```

Expand only the block matching the activity type; remove the others. Pull specific values (scope, path, table name, scenarios) from the PRD row notes and from `.github/context/CONTEXT-SNAPSHOT.md`.

---

### Step 4 — Create Tickets on Redmine

#### 4a — Create module (parent) tickets first

For each module without a parent ticket, run:

```sh
set -a && source .env && set +a  # loads and exports REDMINE_API_KEY
curl -s -X POST \
  -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"issue": {"project_id": "$PROJECT_ID", "subject": "[M{n}] {title}", "description": "...", "tracker_id": $TRACKER_ID_EPICA, "status_id": $STATUS_ID_OPEN}}' \
  "REDMINE_BASE_URL/issues.json"
```

After each call:
1. Parse `issue.id` from the JSON response → `MODULE_TICKET_ID`
2. Record: `M{n} → #MODULE_TICKET_ID`
3. On error: report, skip the module and all its activities, continue

#### 4b — Create activity (child) tickets

For each activity without a ticket, run:

```sh
curl -s -X POST \
  -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"issue": {"project_id": "$PROJECT_ID", "subject": "[M{n}-{nn}] {desc}", "description": "...", "tracker_id": $TRACKER_ID_TASK, "status_id": $STATUS_ID_OPEN, "parent_issue_id": MODULE_TICKET_ID}}' \
  "REDMINE_BASE_URL/issues.json"
```

After each call:
1. Parse `issue.id` → `ACTIVITY_TICKET_ID`
2. Record: `M{n}-{nn} → #ACTIVITY_TICKET_ID`
3. On error: report, skip this activity, continue

> **Process tickets one at a time** — do not batch. Wait for each response before proceeding.

---

### Step 5 — Update PRD.md

#### 5a — Annotate module headings with the parent ticket

Append the ticket reference to the module heading line:

```markdown
### M1 — Infrastructure & Foundation `#456`
```

#### 5b — Annotate activity rows with the child ticket

Embed the reference in the ID cell of each table row:

**Before:**
```markdown
| M1-01 | Audit and complete existing SQL migrations (001–004) | Verify all entities... |
```

**After:**
```markdown
| M1-01 `#457` | Audit and complete existing SQL migrations (001–004) | Verify all entities... |
```

Rules:
- Use backtick-wrapped `#[id]` format
- Insert after the activity ID, before the `|` separator
- Do NOT duplicate if a reference already exists
- Do NOT alter any other cell content

---

### Step 6 — Report Results

After all operations, output a summary table:

| PRD Item | Subject | Ticket | Type | Status |
|----------|---------|--------|------|--------|
| M1 | [M1] Infrastructure & Foundation | #456 | Epica (parent) | Created |
| M1-01 | [M1-01] Audit SQL migrations | #457 | Task | Created |
| M1-02 | [M1-02] Define TypeBox schemas | #458 | Task | Created |
| M2-01 | — | — | — | Skipped (already had #99) |

If any creations failed, list them separately with the error details and suggest the user retry manually using the `create-ticket` operation from the provider config.
