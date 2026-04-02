---
applyTo: 'docs/AI-analysis-plan-docs/**/TICKET-DESC-*.md'
description: 'Rules for writing ticket descriptions for <<PROJECT_NAME>>. Apply when creating or updating TICKET-DESC-*.md files.'
---

# Ticket Description Format

## Purpose

Produce a ticket description in the format required by the configured ticket manager, ready to paste or push directly.

> **Output format is provider-defined.** Read `.github/context/providers/ticket-manager.md` → `## Output Format` for the exact markup rules, escaping rules, and validation checklist.

---

## Output File

- Folder: `docs/AI-analysis-plan-docs/{TICKET-ID}/`
- File: `TICKET-DESC-{TICKET-ID}.md`
- One file per ticket

---

## Standard Section Structure

Every ticket description must include these sections (formatted according to the provider's output format):

1. **Title** — ticket title as a top-level heading
2. **Problem Statement** — what needs to be done and why
3. **Business Requirements** — key requirements from the PRD or ticket
4. **Proposed Solution** — summary of approach
5. **Files to Change** — table with file path and change description
6. **Test Scenarios** — table with scenario and expected result
7. **Risks & Considerations** — bullet list of risks
8. **Definition of Done** — checklist of completion criteria

---

## Content Rules

- Write content from the analysis doc — do NOT invent or hallucinate requirements
- Keep each section concise — ticket descriptions are read by humans skimming
- Use the provider's formatting syntax — **never mix formats** (e.g., no Markdown in Jira wiki markup)
- Include file paths in inline code format (provider-specific: backticks for Markdown, `{{…}}` for Jira, `@…@` for Textile)

---

## Validation

After writing, validate against the **Validation Rules** in the provider config's `## Output Format` section. Every provider includes a validation checklist — check every item.
