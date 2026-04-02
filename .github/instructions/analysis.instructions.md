---
applyTo: 'docs/AI-analysis-plan-docs/**'
description: 'Analysis document format and SDD workflow for <<PROJECT_NAME>>. Apply when creating or updating ANALYSIS-*.md files.'
---

# Analysis Document Format

## Purpose

Create structured technical analysis documents for tickets that guide implementation.

**Prerequisites:**
- Read `core.instructions.md` first
- Read `.github/context/PRD.md` — functional requirements (authoritative)
- Read `.github/context/architecture.md` — module structure
- **Ticket manager MCP:** Before calling any ticket manager tool, read `.github/context/providers/ticket-manager.md` and load via `tool_search_tool_regex` using the provider's load pattern. See `copilot-instructions.md`.

---

## Document Structure

Every analysis document must follow this structure:

```markdown
# Analysis: {Short human title}
> **Ticket:** {TICKET-ID} — {ticket title}
> **Branch:** {suggested branch name}
> **Date:** {YYYY-MM-DD}

---

## Problem Statement

{What needs to be done and why. Must reference spec compliance.}

---

## Current Implementation

{Relevant code paths and current behavior}

---

## Proposed Solution

{High-level design approach}

---

## Implementation Plan

### Files to Change

| File | Changes |
|------|---------|
| path/to/File | Description |

### Step-by-Step Plan

1. **Step 1 Title**
   - Detailed action items

2. **Step 2 Title**
   - More details

---

## Testing Strategy

{What tests to write/update. Reference test conventions from context/best-practices/}

---

## Risks & Considerations

{Edge cases, migration concerns, rollback plan}

---

## References

{Links to Jira, MRs, PRD sections, architecture docs}
```

---

## Specification-Driven Development (SDD)

**MANDATORY WORKFLOW ORDER:**

1. **Read Functional Documentation FIRST** — `context/PRD.md`, `context/architecture.md`, and any external specs referenced there
2. **Read Code Implementation SECOND** — search and read relevant source files
3. **Compare** — does the code match the spec? If NO → this is a **BUG**, document it
4. **Identify gaps** — missing behavior, wrong behavior, untested behavior

> Never assume code is correct. When code and spec disagree, the spec wins.

---

## Output Location

`docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md`

For multi-ticket work sharing a folder: `docs/AI-analysis-plan-docs/{TICKET-ID-A}-{TICKET-ID-B}/`
