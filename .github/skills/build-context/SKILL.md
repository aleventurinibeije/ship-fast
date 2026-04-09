---
name: build-context
description: 'Generate or update context/PRD.md and context/CONTEXT-SNAPSHOT.md from raw project documents (meeting notes, functional specs, user stories, technical docs) in docs/. Use when asked to "build context", "generate context from docs", "write PRD from docs", "update context", "synthesize docs".'
---

# Build Context

## When to Use

Use this skill to generate or update the two primary context files from raw project documentation:

- `context/PRD.md` — product requirements, domain model, module activities
- `context/CONTEXT-SNAPSHOT.md` — tech stack, conventions, auth, test commands, env vars

**Input:** Optional `docsPath` (default: `docs/`). All markdown files found recursively are considered.  
**Output:** Written or updated `context/PRD.md` and `context/CONTEXT-SNAPSHOT.md`, plus a gap report listing what could NOT be extracted automatically.

> **No provider needed.** This skill reads local files only. Skip Step 0.

---

## Prerequisites

Read before starting:
- `.github/instructions/prd.instructions.md` — PRD format rules, stable ID conventions, module scope tags
- `.github/context/PRD.md` — existing PRD (if present) for Update mode
- `.github/context/CONTEXT-SNAPSHOT.md` — existing snapshot (if present) for Update mode

---

## Steps

### Step 1 — Inventory and Categorize Source Documents

1. List all `.md` files recursively under `docs/` (or the provided `docsPath`).
2. Read each file in full.
3. Assign each file one or more categories:

| Category | Signals |
|----------|---------|
| **Functional spec** | Module descriptions, acceptance criteria, entity lists, API contracts |
| **User story** | Flows described from user perspective, "As a ... I want ...", screen lists |
| **Meeting notes** | Date headers, attendees, action items, decisions |
| **Technical spec** | Stack choices, DB schema decisions, integration contracts, env var lists |
| **Architecture doc** | Component diagrams, service layouts, invariants |

4. If `docs/` is empty or absent, report: "No source documents found in `docs/`. Add markdown files and re-run." — then stop.

---

### Step 2 — Determine Mode: Create vs Update

**Create mode** (`context/PRD.md` does not exist or is empty):
- Build both files from scratch (Steps 3–6).

**Update mode** (files already exist):
- Read current `context/PRD.md` fully — note all existing module IDs (`M-N`), activity IDs, and entity names.
- Read current `context/CONTEXT-SNAPSHOT.md` fully — note stack entries, conventions already listed.
- In Steps 3–6, only ADD what is missing. Do NOT rewrite sections that are already accurate. Do NOT renumber existing IDs.

---

### Step 3 — Extract Product Requirements → Draft PRD

From **Functional specs**, **User stories**, and **Meeting notes**:

1. **Project overview**: Extract project name, type (SaaS / internal tool / etc.), target users, key constraints.
2. **Domain model**: List every named entity mentioned (e.g., Building, Resident, Package, Shift). For each:
   - Name, short description, key fields mentioned, persistence status (from tech doc if available)
   - Assign status: `implemented` | `in-progress` | `planned`
3. **Modules**: Group activities by business domain. For each module:
   - Module title and scope (one sentence)
   - Activities (endpoint/feature level granularity — not task level)
   - For each activity: short description, source document citation, ticket reference (if found in the text as `#NNN` or `[TICKET-NNN]`)
4. **Out of scope**: Extract any explicit "not in scope", "phase 2", "future" statements.
5. **Non-functional requirements**: Performance, security, scalability, compliance statements.

Write following the structure mandated by `.github/instructions/prd.instructions.md`:
- Module IDs must be `M-1`, `M-2`, ... (stable; do not change in Update mode)
- Activity IDs must be `M-N.K` (stable)
- One activity = one ticket when implemented

---

### Step 4 — Extract Technical Context → Draft CONTEXT-SNAPSHOT

From **Technical specs** and **Architecture docs**:

1. **Tech stack table**: Framework, runtime version, language, DB, auth, package manager, formatter — one row per layer.
2. **Key file locations**: Entry points, route folders, migration folder, plugin folder, test folder, schema folder.
3. **Domain model** (mirror the PRD entity list with DB table names where available).
4. **Auth layers**: Auth provider, multitenancy strategy, RBAC approach.
5. **Mandatory conventions**: List all named conventions found in technical docs (numbered, max 10–12).
6. **TypeScript / language patterns**: Code snippet examples if found in tech docs.
7. **Testing**: Test framework, test types (unit / integration / e2e), coverage targets if stated.

For fields that cannot be extracted from docs, insert a placeholder:
```
| Test commands | ⚠️ MANUAL — add after reading `package.json` or `pom.xml` |
```

---

### Step 5 — Write / Merge Files

**PRD:**
- Create mode: write `context/PRD.md` with full structure.
- Update mode: append new modules and activities only; do NOT renumber existing stable IDs.

**CONTEXT-SNAPSHOT:**
- Create mode: write `context/CONTEXT-SNAPSHOT.md`.
- Update mode: merge new stack entries, new conventions, new entities — do NOT overwrite existing manually-edited entries. Mark additions with `<!-- auto-added: YYYY-MM-DD -->` comment.

Both files must include a generation stamp at the top:
```
> Auto-generated by `build-context` skill on YYYY-MM-DD. Manual edits are preserved in Update mode.
```

---

### Step 6 — Gap Report

After writing both files, produce a gap report in the chat (do NOT write it to a file):

```
## Context Gap Report

### What was extracted automatically ✓
- [list what was found and written]

### What needs manual fill-in ⚠️
- context/CONTEXT-SNAPSHOT.md → Test commands (read package.json / pom.xml)
- context/CONTEXT-SNAPSHOT.md → Env vars (read .env.example or watt.json)
- context/integrations.md → External API URLs and keys
- context/architecture.md → Service diagrams and Architectural Invariants
- context/providers/ticket-manager.md → If not yet configured, run @bootstrap
- [any other specific gaps found]

### Documents that could not be categorized
- [list any files skipped and why]
```
