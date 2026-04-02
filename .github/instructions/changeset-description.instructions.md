---
description: 'Rules for writing changeset (MR/PR) descriptions for <<PROJECT_NAME>>. Apply when creating CHANGESET-DESC-*.md files or writing changeset descriptions.'
applyTo: 'docs/AI-analysis-plan-docs/**/CHANGESET-DESC-*.md'
---

# Changeset Description Format

## Purpose

Produce a changeset (MR/PR) description ready to paste into the code review platform.

> **Output format is provider-defined.** Read `.github/context/providers/code-review.md` → `## Output Format` for the exact markup rules and validation checklist.

---

## Output File

- Folder: `docs/AI-analysis-plan-docs/{TICKET-ID}/`
- File: `CHANGESET-DESC-{TICKET-ID}.md`
- One file per ticket (if one changeset covers multiple tickets, name after primary ticket)

---

## Standard Changeset Description Structure

```markdown
## Summary

{One paragraph: what this changeset does and why. Reference the ticket(s) with links.}

Closes [{TICKET-ID}]({TICKET_MANAGER_BASE_URL}/browse/{TICKET-ID})

---

## What Changed

{High-level description of the approach, 2–5 sentences. NOT a file list.}

---

## Files Changed

| File | Change |
|---|---|
| `path/to/File` | **NEW** — description |
| `path/to/Other` | **MODIFY** — description |
| `path/to/Old` | **DELETE** — replaced by X |

---

## How to Test

1. Step one
2. Step two
3. Expected result

**Test scenarios:**

| Scenario | Expected Result |
|---|---|
| Happy path | ... |
| Error case | ... |

---

## Checklist

- [ ] Code follows project conventions
- [ ] Unit tests added / updated
- [ ] All tests pass
- [ ] No breaking changes to existing behaviour

---

## Notes for Reviewer

{Optional: design decisions, trade-offs, known limitations, follow-up tickets. Omit section if nothing to add.}
```

---

## Content Rules

- Format output according to the code-review provider's `## Output Format` rules
- Inline code (in the provider's syntax) for file names, class names, config keys
- Keep each section concise — reviewers skim
- "What Changed" describes approach and decisions, not a file list
- "Files Changed" is the exhaustive file list — include every modified file
- Build ticket links using the ticket-manager provider's Constants → Base URL

---

## Validation

After writing, validate against the **Validation Rules** in the code-review provider's `## Output Format` section.
