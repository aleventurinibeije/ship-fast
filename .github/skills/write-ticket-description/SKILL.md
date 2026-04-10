---
name: write-ticket-description
description: 'Write ticket descriptions in the format required by the configured ticket manager. Use when asked to "write ticket description", "write description", "convert analysis to ticket format", or given a ticket number.'
---

# Write Ticket Description

## When to Use

Use this skill to produce a ticket description in the format required by the configured ticket manager (Jira, Redmine, GitHub Issues, etc.).

**Input:** Ticket number(s) or explicit analysis document path
**Output:** `docs/AI-analysis-plan-docs/{TICKET-ID}/TICKET-DESC-{TICKET-ID}.md` — one file per ticket

---

## Prerequisites

Read before writing:
- `.github/instructions/ticket-description.instructions.md` — generic structure and content rules
- `.github/context/providers/ticket-manager.md` — **REQUIRED:** MCP tools, operations, output format rules, constants

---

## Steps

### Step 0 — Load Provider (if configured)

Follow the **Standard Provider Loading Steps** from `.github/context/providers/LOADING-PROTOCOL.md` for the **ticket-manager** provider (`.github/context/providers/ticket-manager.md`).

Extract and store the **Constants** and **Output Format** sections for use in all subsequent steps.

---

### Step 1 — Identify Tickets

Scan user input for the ticket pattern from the provider config's `## Constants` → `Ticket pattern`.
If multiple tickets, produce a separate file for each.

### Step 2 — Find the Source

For each ticket, check in order:

1. **Existing analysis doc:** `docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md`
2. **Ticket manager MCP** (if no analysis doc and provider is configured):
   Call the **fetch-ticket** operation from the provider config.
   Extract: title, description, comments.

### Step 3 — Read the Analysis Doc

Extract from the analysis document:
- Problem statement
- Business requirements
- Proposed solution
- Files to change (table)
- Test scenarios (table)
- Risks and considerations
- Definition of Done

### Step 4 — Convert to Provider Output Format

Read the `## Output Format` section from the provider config and apply **all** its formatting rules.

The output format varies by provider:
- **Jira:** Jira wiki markup (headings `h1.`, lists `* item`, code `{code:lang}…{code}`)
- **Redmine:** Markdown (headings `##`, code ` ```lang … ``` `)
- **GitHub Issues:** GitHub-flavored Markdown
- **Others:** Follow the rules documented in the provider config

### Step 5 — Write the File

Write to `docs/AI-analysis-plan-docs/{FOLDER}/TICKET-DESC-{TICKET-ID}.md`.

If tickets share a folder, write both files into that same folder. Do NOT create new folders.

### Step 6 — Validate

Re-read the written file and check against the **Validation Rules** in the provider config's `## Output Format` section.

### Step 7 — Optional: Push to Ticket Manager

If the user asks to push, call the **update-ticket** operation from the provider config with the formatted content.
