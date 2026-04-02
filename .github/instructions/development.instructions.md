---
applyTo: 'src/**'
description: 'Development patterns, conventions, and testing practices for <<PROJECT_NAME>>. Active on all source files.'
---

# <<PROJECT_NAME>> — Development Patterns

**Prerequisites:** Read `core.instructions.md` first.  
**Business rules:** See `.github/context/PRD.md` — authoritative; overrides code when they conflict.

---

## Mandatory Conventions

Read `.github/context/best-practices/backend.md` and/or `.github/context/best-practices/frontend.md` for the full rule set. These files are the authoritative coding standards.

**Key invariants from architecture (always enforced):**  
See `.github/context/architecture.md` → `## Architectural Invariants` section.

---

## Implementation Workflow

1. Read the analysis doc (if exists): `docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md`
2. Read `context/best-practices/backend.md` or `frontend.md` for the relevant stack
3. Read target files before editing
4. Implement following the recipes in the best-practices file
5. Write tests following the test conventions section of the best-practices file
6. Use `get_errors` to validate compilation after each change
7. Update analysis doc — mark implementation sections complete

---

## Testing

Testing conventions (frameworks, naming, structure, coverage targets) are defined in:
- `.github/context/best-practices/backend.md` — BE test conventions
- `.github/context/best-practices/frontend.md` — FE test conventions

Always follow the test class structure and arrange/act/assert pattern specified there.
