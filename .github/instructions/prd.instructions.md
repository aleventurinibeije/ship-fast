---
applyTo: '.github/context/PRD.md'
description: 'PRD format, update rules, and source-folder conventions for <<PROJECT_NAME>>. Apply when creating or updating PRD.md.'
---

# PRD Format & Update Rules

## Purpose

`.github/context/PRD.md` is the **authoritative product requirements document** for the backend project. It is the single source of truth that all other tools (analysis, dev, tickets) read first.

**Source folders:**

| Folder | Role |
|--------|------|
| `docs/analysis/` | Primary functional specification (technical attachments, spec documents) |
| `docs/user-stories/` | Optional — user stories with flows, constraints, and backend service lists |

Both folders can grow incrementally. When new files are added, run the `write-prd` skill to integrate them.

---

## Invariants

1. **Backend only** — the PRD describes server-side behaviour (endpoints, migrations, business logic). No UI, no component names, no CSS.
2. **Specification-driven** — if the code contradicts the PRD, the code is wrong. The analysis docs override the code.
3. **Never renumber IDs** — `M{N}` module IDs and `M{N}-NN` activity IDs are stable once written. New items append at the end.
4. **One ticket per activity** — each row in an activity table must be independently implementable and ticketable.
5. **Source citations** — every module enriched by a user story must carry `> **Source:** docs/user-stories/US-XYZ.md`.

---

## Update Protocol

When new source files arrive (new analysis doc or user story):

1. Read the new file(s) in full.
2. Read the current PRD in full.
3. **Diff** — identify what is absent from the PRD:
   - Missing entities → add to Domain Model table with status `❌ not yet defined`.
   - Missing endpoints → add as activities to the matching module (or create a new module).
   - Missing constraints or business rules → annotate the relevant module header.
   - Missing migrations → add a `Migration:` activity and update entity status.
4. Append only. Do not rewrite accurate existing sections.
5. Update the `> **Version:**` line with the new date.

---

## Domain Model Status Values

| Status | Meaning |
|--------|---------|
| `✅ migrated` | SQL migration exists and has been applied |
| `⚠️ partial` | Table exists but is missing columns added by a subsequent user story |
| `❌ not yet defined` | No migration yet; entity is planned |

---

## Module Scope Tags

Each module header should indicate the primary consumer role:

- `(admin)` — only admin users can call these endpoints
- `(staff)` — only staff (guardian/vigilance) users
- `(resident)` — only resident users
- `(mixed)` — module contains endpoints for multiple roles; each activity note must specify the role

---

## Non-Negotiable Activity Notes

Every activity row **must** have a non-empty Notes column covering at least one of:

- Target role (`admin scope required`, `staff only`, `resident`)
- Key constraint or business rule
- Migration or dependency reference
- Link to another activity (`see M8-07`)
