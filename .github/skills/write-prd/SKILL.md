---
name: write-prd
description: 'Create or update the project PRD from analysis documents and optional user stories. Reads all files in docs/analysis/, optionally integrates docs/user-stories/, and writes/updates .github/context/PRD.md. Use when asked to "write PRD", "create PRD", "update PRD", "integrate user stories into PRD".'
---

# Write PRD

## When to Use

Use this skill to:
1. **Create** `.github/context/PRD.md` from scratch using analysis documents and optional user stories.
2. **Update** an existing PRD by integrating new analysis documents or newly added user stories that are not yet reflected.

**Input:** Optional scope hint (e.g., "only integrate user stories", "rebuild from scratch", "add the new analysis doc")  
**Output:** Written or updated `.github/context/PRD.md`

---

## Source Folders

| Folder | Role | Required |
|--------|------|----------|
| `docs/analysis/` | Primary input — functional specs, technical attachments, feature descriptions | Yes (at least one file) |
| `docs/user-stories/` | Secondary input — user stories with flows, constraints, backend service lists | No — integrate only if files exist |

Both folders can receive new files at any time. Running this skill again will detect and integrate what is missing.

---

## Prerequisites

Read before starting:
- `.github/context/architecture.md` — domain model, tech stack, module layout, invariants
- `.github/context/PRD.md` — existing PRD (if present) for diff/update mode
- All files under `docs/analysis/` — authoritative functional specification
- All files under `docs/user-stories/` (if the folder exists and is non-empty)

---

## Steps

### Step 1 — Inventory Source Files

1. List `docs/analysis/` — collect all `.md` files.
2. List `docs/user-stories/` — collect all `.md` files (may be empty or absent).
3. Read each file in full.
4. For each user story, extract:
   - Functional description
   - Workflow / mermaid diagram
   - UI/UX screens (for BE context: identify which screens require backend endpoints)
   - Constraints and business rules
   - **Backend services list** (Section "Servizi di Backend" or equivalent) — these map directly to PRD activities
   - DB fields to persist

---

### Step 2 — Determine Mode: Create vs Update

**Create mode** (PRD is empty or does not exist):
- Build the full PRD from scratch following the [PRD Structure](#prd-structure) below.

**Update mode** (PRD already exists):
- Read the current PRD fully.
- For each analysis doc: check if its content is already represented in the PRD (compare module titles, entity list, activity IDs).
- For each user story: check if the story's backend services and constraints are already captured.
- Only add what is **missing**:
  - New entities → add to Domain Model table
  - New backend endpoints → add as activities to the relevant module, or create a new module
  - New constraints → annotate the relevant module header
  - New migrations → mark entity status
- Add a `> **Source:** docs/user-stories/US-XYZ.md` citation on the module that the story enriches.
- Do **not** rewrite sections that are already accurate.

---

### Step 3 — Map User Stories to Modules

For each user story, identify which PRD module it belongs to by matching:
- The story's domain (auth, packages, keys, shifts, booking, etc.) → existing module ID
- If no module matches → create a new module (next available `M-N` ID)

For each matched module:
- Add missing activities (endpoints, migrations, utilities).
- Enrich the module description with constraints from the story.
- Add the `> **Source:**` citation.

---

### Step 4 — Write / Update PRD.md

Follow the [PRD Structure](#prd-structure) exactly. Then validate:

- [ ] Every domain entity in `architecture.md` appears in the Domain Model table.
- [ ] Every backend service listed in user stories appears as a PRD activity.
- [ ] Every user story constraint is captured either in a module header or an activity note.
- [ ] Activities that introduce new DB tables have a corresponding `Migration:` activity entry.
- [ ] `Out of Scope` section lists any features from source docs explicitly deferred.
- [ ] Non-Functional Requirements section is complete.

---

## PRD Structure

Every PRD must follow this exact structure:

```markdown
# {ProjectName} BE — Product Requirements Document

> **Scope:** Backend only. ...
> **Reference:** docs/analysis/{main-spec-file} — authoritative functional spec.
> **Version:** {N.N} — {date}

---

## Project Overview

{2–4 paragraph description: what the system does, who uses it, the three main roles, architecture summary}

### Core Roles

| Role | Description |
|------|-------------|
| ...  | ...         |

### Architecture Summary

- **Runtime:** ...
- **Framework:** ...
- ...

---

## Domain Model

| Entity | Table | Status |
|--------|-------|--------|
| `entityName` | `table_name` | ✅ migrated / ❌ not yet defined |

---

## Modules & Granular Activities

{One H3 section per module, ID format: M1, M2, ..., MN}

### M{N} — {Module Title}

> {One-sentence module description. Business rationale.}
>
> **Source:** `docs/user-stories/US-XYZ.md`   ← only if a user story was integrated

{Optional: constraints block using bullet list}

| ID | Activity | Notes |
|----|----------|-------|
| M{N}-01 | {verb + endpoint or migration or utility} | {scope, edge cases, links} |
| ...      | ...                                        | ...                         |

---

## Out of Scope (Future)

{Bulleted list of features mentioned in source docs but explicitly excluded from this version}

---

## Non-Functional Requirements

| Area | Requirement |
|------|-------------|
| Security | ... |
| Error format | ... |
| ...  | ...  |
```

---

## Activity Naming Conventions

- **Endpoints:** `\`{METHOD} {/path}\` — {short description}`
- **Migrations:** `Migration: {describe table/column change}`
- **Utilities / helpers:** `Implement {name} — {purpose}`
- **Tests:** `Write integration tests for {module/route group}`
- Each activity must be independently ticketable (single endpoint, migration, or test suite per activity).

---

## ID Assignment Rules

- Module IDs: `M1`, `M2`, … assigned in order of dependency (foundation first, features after).
- Activity IDs: `M{N}-01`, `M{N}-02`, … sequential within a module.
- When updating an existing PRD, **never renumber** existing IDs. Append new activities at the end of the relevant module.

---

## Quality Rules

- Every activity must have a non-empty Notes column (scope, target role, edge case, or link).
- Module headers must state whether the module is `admin`, `staff`, `resident`, or mixed scope.
- If a user story provides a mermaid flow, summarise the flow as business rules in the module description — do not paste raw diagrams into the PRD.
- Activities must be written from a **backend perspective** (endpoints, migrations, business logic) — no UI details.
