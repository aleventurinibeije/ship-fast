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
- `.github/context/architecture.md`
- `.github/context/providers/ticket-manager.md` — MCP tools, operations, constants

---

## Steps

### Step 0 — Load Ticket Manager Provider (MANDATORY FIRST STEP)

1. Read `.github/context/providers/ticket-manager.md`
2. If the file is a placeholder (contains only HTML comments), **no MCP provider is configured** — skip MCP steps and ask the user for ticket details manually
3. If configured, extract the **Load pattern** from `## MCP Loading` and call:

```
tool_search_tool_regex
  pattern: {load_pattern_from_provider}
```

Wait for results. Only after this call completes can you call any provider tools.

4. Extract **Constants** (project key, ticket pattern, etc.) for use in subsequent steps.

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

### Step 3 — Determine Create vs Update

**Create:** `docs/AI-analysis-plan-docs/{TICKET-ID}/` does not exist  
**Update:** Folder exists — read existing `ANALYSIS-{TICKET-ID}.md` before modifying

### Step 4 — Gather Context (Specification-Driven Order)

**MANDATORY: Read in this order.**

**Phase 1: Functional Documentation — read in PARALLEL**

1. `.github/context/PRD.md` — authoritative product requirements
2. `.github/context/architecture.md` — module structure and invariants
3. Any external specs referenced in `PRD.md` for the relevant feature area

Search each for keywords from the ticket. Extract the TRUE requirements.

**Phase 2: Code Implementation — read AFTER Phase 1**

Search and read source files relevant to the ticket. For every implementation detail, compare against Phase 1 specs.

> **Discrepancies between code and spec are BUGS — not "how it works".**

**Phase 3: Validation**
- Does code match `PRD.md`? If NO → BUG, document it
- Do tests validate correct behavior per spec? If NO → tests are wrong
- Are there gaps between spec and implementation?

### Step 5 — Draft Analysis Document

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

### Step 6 — Create or Update File

Write to `docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md`.

For multi-ticket work: use a combined folder `{TICKET-A}-{TICKET-B}/` if they share implementation.

### Step 7 — Validate

- [ ] Problem Statement references spec (cites `PRD.md` or external spec)
- [ ] Bugs found during Phase 3 are documented in Risks or Problem Statement
- [ ] Files to Change table is complete (no missing files)
- [ ] Testing Strategy references `context/best-practices/backend.md` or `frontend.md`
