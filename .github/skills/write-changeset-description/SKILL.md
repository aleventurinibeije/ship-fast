---
name: write-changeset-description
description: 'Write a changeset (MR/PR) description in the format required by the configured code-review platform. Use when asked to "write MR description", "write PR description", "write changeset description", or given a ticket number.'
---

# Write Changeset Description

## When to Use

Use this skill to produce a changeset (MR/PR) description for one or more tickets.

**Input:** Ticket number(s) or changeset URL
**Output:** `docs/AI-analysis-plan-docs/{TICKET-ID}/CHANGESET-DESC-{TICKET-ID}.md` — one file per ticket (or covering multiple if they share a changeset)

---

## Prerequisites

Read before writing:
- `.github/instructions/changeset-description.instructions.md` — generic structure, content rules, validation
- `.github/context/providers/code-review.md` — **REQUIRED:** MCP tools, operations, output format, constants
- `.github/context/providers/ticket-manager.md` — for ticket links and context (optional)

---

## Steps

### Step 0 — Load Providers (if needed)

1. Read `.github/context/providers/code-review.md`
2. If a changeset URL is provided and the provider is configured, extract the **Load pattern** and call:

```
tool_search_tool_regex
  pattern: {load_pattern_from_code_review_provider}
```

3. Read `.github/context/providers/ticket-manager.md`
4. If ticket context is needed and the provider is configured, extract its **Load pattern** and call:

```
tool_search_tool_regex
  pattern: {load_pattern_from_ticket_manager_provider}
```

Both can be called in parallel if both are needed.

---

### Step 1 — Identify Tickets and Changeset

Scan user input for ticket pattern (from ticket-manager provider → Constants → `Ticket pattern`).
If a changeset URL is provided, extract the changeset ID.

### Step 2 — Gather Source Material

Collect in priority order:

**1. Existing analysis doc:**
```
docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md
```

**2. Git log:**
```bash
git log --oneline origin/main..HEAD
git diff --name-status origin/main..HEAD
git diff --stat origin/main..HEAD
```

**3. Code review platform API** (if URL provided and provider configured):
Call the **fetch-changeset** and **fetch-changeset-diff** operations from the code-review provider.

**4. Ticket manager API** (for business context, if provider configured):
Call the **fetch-ticket** operation from the ticket-manager provider.

### Step 3 — Read Changed Files

For each modified file in the diff, read its content to understand what was actually implemented. The description must reflect reality, not just the analysis plan.

### Step 4 — Write Changeset Description

Read the `## Output Format` section from the code-review provider config.
Build ticket link using the ticket-manager provider's Constants → Base URL and ticket ID.

Structure:
1. `## Summary` — what + why + ticket link
2. `## What Changed` — approach and key decisions (not a file list)
3. `## Files Changed` — table of every modified file
4. `## How to Test` — numbered steps + test scenarios table
5. `## Checklist` — standard + ticket-specific items
6. `## Notes for Reviewer` — design decisions, trade-offs (omit if nothing to add)

### Step 5 — Write the File

Write to `docs/AI-analysis-plan-docs/{FOLDER}/CHANGESET-DESC-{TICKET-ID}.md`.

### Step 6 — Validate

Check against the **Validation Rules** in the code-review provider's `## Output Format` section.

### Step 7 — Optional: Push to Code Review Platform

If the user asks to update the changeset description, call the **update-changeset** operation from the code-review provider config.
