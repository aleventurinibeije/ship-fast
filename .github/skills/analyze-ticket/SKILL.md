---
name: analyze-ticket
description: 'Create or update a technical analysis document for a ticket. Fetches ticket details via the configured ticket-manager provider, reads context/PRD.md and context/architecture.md, explores the codebase, and produces a structured analysis + development plan.'
---

# Analyze Ticket

## When to Use

Use this skill when you need to create or update a technical analysis document for a development ticket.

**Input:** Ticket number, ticket URL, or description (e.g. "MYAPP-123", "https://your-tracker.example.com/browse/MYAPP-123")
**Output:** Structured analysis document in `docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md`

---

## Prerequisites

Read before starting:
- `.github/instructions/core.instructions.md`
- `.github/instructions/analysis.instructions.md`
- `.github/context/PRD.md`
- `.github/context/CONTEXT-SNAPSHOT.md` — tech stack, module layout, auth layers, invariants (replaces architecture.md + best-practices/backend.md)
- `.github/context/providers/ticket-manager.md` — MCP tools, operations, constants
- `.github/context/providers/LOADING-PROTOCOL.md` — provider loading steps (use for Step 0)

---

## Steps

### Step 0 — Load Ticket Manager Provider (MANDATORY FIRST STEP)

Follow the **Standard Provider Loading Steps** from `.github/context/providers/LOADING-PROTOCOL.md` for the **ticket-manager** provider (`.github/context/providers/ticket-manager.md`).

Extract and store the **Constants** (project key, ticket pattern, status IDs) for use in all subsequent steps.

---

### Step 1 — Extract Ticket ID

Scan user input for the project ticket pattern (from ticket-manager provider → Constants → `Ticket pattern`).
Example: `MYAPP-\d+` → `MYAPP-123`

If no ticket ID found, use a descriptive folder name (e.g., `MISC-feature-name`).

### Step 2 — Fetch Ticket

Call the **fetch-ticket** operation from the ticket-manager provider config, using the tool name and parameters documented there.

Extract from the response (field names vary by provider — refer to the provider's **Response fields**):
- Ticket title
- Full description
- Issue type (Story / Bug / Task)
- Current status
- Assignee
- Recent comments
- Linked issues

### Step 3 — Update Ticket Status

If a ticket was successfully fetched, use the **update-ticket** operation from `context/providers/ticket-manager.md` to set its status to **In Analysis** (`status_id_in_analysis` from provider Constants — skip if the constant is not defined).

### Step 4 — Determine Create vs Update

**Create:** `docs/AI-analysis-plan-docs/{TICKET-ID}/` does not exist  
**Update:** Folder exists — read existing `ANALYSIS-{TICKET-ID}.md` before modifying

### Step 5 — Gather Context (Specification-Driven Order)

**MANDATORY: Read in this order.**

**Phase 1: Functional Documentation — read in PARALLEL**

1. `.github/context/PRD.md` — authoritative product requirements
2. `.github/context/CONTEXT-SNAPSHOT.md` — tech stack, module layout, invariants (already loaded if following Prerequisites)
3. Any external specs referenced in `PRD.md` for the relevant feature area

Search each for keywords from the ticket. Extract the TRUE requirements.

**Phase 2: Code Implementation — read AFTER Phase 1**

Search and read source files relevant to the ticket. For every implementation detail, compare against Phase 1 specs.

> **Discrepancies between code and spec are BUGS — not "how it works".**

**Phase 3: Validation**
- Does code match `PRD.md`? If NO → BUG, document it
- Do tests validate correct behavior per spec? If NO → tests are wrong
- Are there gaps between spec and implementation?

### Step 6 — Draft Analysis Document

Follow structure from `analysis.instructions.md`:

```markdown
# Analysis: {Short human title}
> **Ticket:** {TICKET-ID} — {ticket title}
> **Branch:** {suggested branch name}
> **Date:** {YYYY-MM-DD}

## Problem Statement
## Current Implementation
## Proposed Solution
## Implementation Plan
  ### Files to Change (table)
  ### Step-by-Step Plan
## Testing Strategy
## Risks & Considerations
## References
```

### Step 7 — Create or Update File

Write to `docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md`.

For multi-ticket work: use a combined folder `{TICKET-A}-{TICKET-B}/` if they share implementation.

### Step 8 — Post Analysis Comment to Ticket

After writing the analysis file, call the **update-ticket** operation from `context/providers/ticket-manager.md` to add a comment to the Redmine ticket with the key sections of the analysis.

Build the `notes` value as follows. **All content must be in Textile markup** (see `context/providers/ticket-manager.md` → Output Format). Content copied from the Markdown analysis file must be converted to Textile — no `##`, no backticks, no `**bold**`.

```textile
*[AI Analysis]* — {TICKET-ID}: {ticket title}
*Branch:* {suggested branch name}

----

h2. Problem Statement

{full content of the Problem Statement section, converted to Textile}

----

h2. Proposed Solution

{full content of the Proposed Solution section, converted to Textile}

----

h2. Implementation Plan

{full content of the Step-by-Step Plan sub-section, including step titles and all action items, converted to Textile}

----

Analysis document: @docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md@
```

Use `curl` via `run_in_terminal` as documented in `context/providers/ticket-manager.md` → **update-ticket**, passing `notes` in the request body.

> This step runs for both **create** and **update** cases. If the ticket was not found in Step 2, skip this step.

### Step 9 — Validate

- [ ] Problem Statement references spec (cites `PRD.md` or external spec)
- [ ] Bugs found during Phase 3 are documented in Risks or Problem Statement
- [ ] Files to Change table is complete (no missing files)
- [ ] Testing Strategy references `context/best-practices/backend.md` or `frontend.md`
