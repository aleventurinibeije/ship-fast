---
name: review-changeset
description: 'Review a changeset (MR/PR). Fetches metadata, diffs, and discussions via the configured code-review provider, performs structured code review against project conventions, and produces a review document.'
---

# Review Changeset

## When to Use

Use this skill to review a changeset (merge request / pull request) from a URL.

**Input:** Changeset URL (e.g. `https://gitlab.example.com/.../merge_requests/42` or `https://github.com/.../pull/42`)
**Output:** Structured review document

---

## Prerequisites

Read before starting:
- `.github/instructions/core.instructions.md`
- `.github/instructions/review.instructions.md`
- `.github/context/architecture.md`
- `.github/context/providers/code-review.md` — **REQUIRED:** MCP tools, operations, constants

---

## Steps

### Step 0 — Load Provider and MCP Tools (MANDATORY FIRST STEP)

1. Read `.github/context/providers/code-review.md`
2. Extract the **Load pattern** from the `## MCP Loading` section
3. Call `tool_search_tool_regex` with that pattern

```
tool_search_tool_regex
  pattern: {load_pattern_from_provider}
```

Wait for results. Do NOT proceed until complete.

4. Extract the **Constants** (project ID, etc.) and store them for use in subsequent steps.

---

### Step 1 — Parse Changeset URL and Read Constants

From the changeset URL, extract the changeset ID (integer at end of URL).

Read the project identifier from the provider config's `## Constants` section.

### Step 2 — Fetch Changeset Metadata

Call the **fetch-changeset** operation from the provider config:
- Use the tool name, parameter names, and constants documented there

Extract: title, description, source branch, target branch, state, author, labels.

**Extract ticket ID:** Scan `source_branch`, `title`, then `description` for the project ticket pattern (from `context/providers/ticket-manager.md` → Constants → Ticket pattern, if that provider is configured).

### Step 3 — Fetch Changeset Diffs

Call the **fetch-changeset-diff** operation from the provider config.

List all changed files. Group by: new / modified / deleted.

### Step 4 — Fetch Existing Discussions

Call the **fetch-changeset-discussions** operation from the provider config.

Note already-raised concerns to avoid duplicating them.

### Step 5 — Load Local Context

1. If a ticket ID was found, check for analysis doc: `docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md`
2. Read all changed source files from the diffs
3. Read coupled dependencies (files imported/called by the changed files)
4. Re-read `.github/context/best-practices/backend.md` or `frontend.md` as appropriate

### Step 6 — Apply Review Checklist

Run through all categories in `review.instructions.md`:
1. Architecture Compliance (checked against `context/architecture.md` invariants)
2. Database Changes (if applicable)
3. Testing
4. Code Quality
5. Frontend-specific (if applicable)

For each finding, note: **Critical** (must fix) / **Major** (should fix) / **Minor** (suggestion).

### Step 7 — Generate Review Document

Follow the output format from `review.instructions.md`. Save to:
`docs/AI-analysis-plan-docs/{TICKET-ID}/REVIEW-{CHANGESET-ID}.md`

If no ticket ID was identified, use: `docs/AI-analysis-plan-docs/REVIEW-{CHANGESET-ID}.md`

### Step 8 — Optional: Post to Code Review Platform

If the user asks to post comments, call the **post-changeset-comment** operation from the provider config.

Post critical and major findings only. Skip nits unless requested.
