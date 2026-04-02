---
name: create-tickets-from-prd
description: 'Scan PRD.md for development items that do not yet have a ticket reference, create Redmine tickets for each one, and update PRD.md with the ticket numbers. Use when asked to "create tickets from PRD", "sync PRD with tracker", "ticket PRD items", "generate tickets".'
---

# Create Tickets from PRD

## When to Use

Use this skill to:
1. Scan `.github/context/PRD.md` for Features / User Stories / acceptance criteria that have **no ticket reference** (`#\d+`)
2. Create a Redmine ticket for each item without a reference
3. Update `PRD.md` in-place to embed the ticket reference next to each item

**Input:** Optional scope hint (e.g., "only F1", "user story level", "API tasks")
**Output:** Updated `.github/context/PRD.md` with `#[id]` references; created tickets on Redmine

---

## Prerequisites

Read before starting:
- `.github/context/PRD.md` — the authoritative source of development work
- `.github/context/providers/ticket-manager.md` — Redmine API, constants, output format rules

---

## Steps

### Step 0 — Read Provider Config

1. Read `.github/context/providers/ticket-manager.md`
2. Extract from **Constants**:
   - `Base URL` → `REDMINE_BASE_URL`
   - `API key env var` → `REDMINE_API_KEY`
   - `Project identifier` → `PROJECT_ID`
   - `Ticket pattern` → regex for detecting existing references (e.g. `#\d+`)
3. Note the `create-ticket` operation parameters (endpoint, body fields, response fields)

> **Security:** Never hardcode the API key. Use it as a shell variable:
> `REDMINE_API_KEY=<value from Constants>`

---

### Step 1 — Read and Parse PRD

1. Read `.github/context/PRD.md` fully
2. Build an **inventory** of ticketable items at two possible granularity levels:

   **Feature level** (coarse — one ticket per top-level feature):
   ```
   ### F1 — <title>
   ### F2 — <title>
   ```

   **User Story level** (default — one ticket per User Story):
   ```
   ### US-<id> — <title>
   ```

   **Task level** (fine — one ticket per sub-item, e.g. per backend endpoint or acceptance criterion):
   Use only when the user explicitly requests this granularity.

3. For each item, check whether a ticket reference matching the ticket pattern (`#\d+`) already appears **on the same line as the heading or in the first 3 lines of the section**.

4. Build two lists:
   - **WITH reference** → skip, already tracked
   - **WITHOUT reference** → queue for ticket creation

5. If **all items already have references**, report this to the user and stop.

---

### Step 2 — Confirm Scope with User (if ambiguous)

If the PRD has items at multiple granularity levels and the user did not specify, show the pending list and ask:

> "I found N items without a ticket reference. I'll create tickets at **User Story level** by default. Should I proceed, or do you prefer a different granularity (Feature / Task)?"

Proceed immediately if the intent is clear.

---

### Step 3 — Prepare Ticket Data

For each item in the **WITHOUT reference** list, prepare:

| Field | Value |
|-------|-------|
| `subject` | Item title (strip leading `###`, `US-`, `F1 —` etc., keep it human-readable) |
| `description` | Full section content from PRD, converted to Textile markup (see Output Format in provider config) |
| `tracker_id` | `2` (Feature) for US/Feature items; `3` (Task) for API endpoints or sub-tasks |
| `project_id` | `$PROJECT_ID` (from provider constants — Step 0) |

**Description content rules:**
- Include the Acceptance Criteria / Flusso sections from the PRD section
- Include relevant tables (Schermate, API, DB fields) in Textile table format
- Do NOT use Markdown syntax — use Textile only (see provider Output Format)
- Keep descriptions concise but complete

---

### Step 4 — Create Tickets on Redmine

For each item, run the `create-ticket` curl from the provider config:

```sh
REDMINE_API_KEY=<value>
curl -s -X POST \
  -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"issue\": {\"project_id\": \"$PROJECT_ID\", \"subject\": \"...\", \"description\": \"...\", \"tracker_id\": 2}}" \
  "$REDMINE_BASE_URL/issues.json"
```

After each call:
1. Parse `issue.id` from the JSON response
2. Record the mapping: `PRD heading → #[issue.id]`
3. If the response indicates an error (no `issue.id` in response), report the failure, skip this item, and continue with the rest

> **Process items one at a time** — do not batch. Wait for each response before proceeding.

---

### Step 5 — Update PRD.md

For each successfully created ticket, update `.github/context/PRD.md`:

**Heading lines** — append the ticket reference after the title on the same line:

```markdown
### US-login_installazione — Gestione Accessi e Installazione PWA `#123`
```

**Feature lines** — same pattern:

```markdown
### F1 — Gestione Accessi e Workflow di Installazione PWA `#123`
```

Rules:
- Use backtick-wrapped `#[id]` format for readability in Markdown
- Append at the end of the heading line, after any existing text
- Do NOT duplicate if a reference already exists
- Do NOT alter any other content in the PRD

---

### Step 6 — Report Results

After all operations, output a summary table:

| PRD Item | Ticket | Status |
|----------|--------|--------|
| US-my-feature | #123 | Created |
| US-another-feature | #124 | Created |
| F1 | — | Skipped (already had #98) |

If any creations failed, list them separately with the error details and suggest the user retry manually using the `create-ticket` operation from the provider config.
