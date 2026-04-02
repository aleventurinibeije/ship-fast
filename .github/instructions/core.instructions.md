---
applyTo: '**/*'
description: 'Core project context for <<PROJECT_NAME>> — architecture, pipeline, and key concepts. Always active.'
---

# <<PROJECT_NAME>> — Core Context

**Business rules and product requirements:** See `.github/context/PRD.md` — authoritative source; overrides code when they conflict.  
**Architecture and module layout:** See `.github/context/architecture.md`.  
**External integrations and constants:** See `.github/context/integrations.md`.  
**MCP provider configs:** See `.github/context/providers/` — ticket manager, code review, design tool.

---

## Mandatory Reading

Before implementing any feature or fix, read:

1. `.github/context/PRD.md` — understand the TRUE requirements
2. `.github/context/architecture.md` — understand the module layout and invariants
3. `.github/context/integrations.md` — project constants (test commands, APIs)
4. `.github/context/providers/` — MCP provider configs (ticket manager, code review, design tool)

> **Spec before code:** If the code contradicts `PRD.md`, the code is wrong. Discrepancies are bugs.

---

## Specification-Driven Development (SDD)

**MANDATORY WORKFLOW ORDER:**

1. Read functional documentation **FIRST** — extract true requirements
2. Read code implementation **SECOND** — compare against requirements
3. Never assume code is correct — discrepancies are bugs, not "how it works"
4. Identify gaps: missing behavior, wrong behavior, untested behavior
