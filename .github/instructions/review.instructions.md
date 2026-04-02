---
description: 'Changeset review checklist for <<PROJECT_NAME>> — code quality, architectural compliance, and project standards. Load when reviewing changesets (MRs/PRs).'
---

# Changeset Review Checklist

## Purpose

Review changesets (merge requests / pull requests) to ensure code quality, architectural compliance, and alignment with project standards.

**Prerequisites:**
- Read `core.instructions.md` first
- Read `.github/context/architecture.md` — architectural invariants
- Read `.github/context/providers/code-review.md` — code review platform operations and constants
- Read `.github/context/best-practices/backend.md` (if changeset touches BE)
- Read `.github/context/best-practices/frontend.md` (if changeset touches FE)

---

## Review Process

1. **Fetch changeset metadata** — title, description, source/target branch, labels
2. **Fetch changeset diffs** — all changed files
3. **Fetch existing discussions** — avoid duplicating comments
4. **Load local context** — read changed files + coupled dependencies
5. **Apply checklist** — run through all categories below
6. **Generate review document** — structured findings

---

## Platform Configuration

**Provider config:** Read from `.github/context/providers/code-review.md`
- Project identifier and MCP tool names are in the provider's `## Constants` and `## Operations` sections
- Load MCP tools using the provider's `## MCP Loading` → Load pattern

---

## Review Categories

### 1. Architecture Compliance

Check against `.github/context/architecture.md` → `## Architectural Invariants`:
- [ ] All stated invariants are respected
- [ ] Module boundaries not violated (services don't reach into other service internals)
- [ ] Dependency direction follows the architecture diagram

#### Dependency Injection (if applicable)
- [ ] Constructor injection used (no field-level `@Autowired` or equivalent)
- [ ] All injected dependencies are final/readonly

#### Configuration (if applicable)
- [ ] Config properties use typed config classes (no magic strings)
- [ ] Defaults provided for optional properties

#### Security
- [ ] Endpoints secured appropriately
- [ ] No sensitive data logged
- [ ] No stack traces in API responses
- [ ] Input validated at system boundaries

---

### 2. Database Changes (if applicable)

- [ ] Schema changes use migration tool (see `context/architecture.md`)
- [ ] Migration is idempotent (safe to re-run)
- [ ] Migration naming follows project convention
- [ ] Entity and model classes updated consistently
- [ ] Mapper/DTO updated to include new fields

---

### 3. Testing

- [ ] Coverage meets targets defined in `context/best-practices/backend.md` or `frontend.md`
- [ ] Test naming follows project convention
- [ ] Tests use arrange/act/assert structure
- [ ] Tests cover happy path AND error/edge cases
- [ ] Integration tests updated for new flows (if applicable)
- [ ] No tests that only test framework behavior (mocks without assertions)

---

### 4. Code Quality

- [ ] No duplication — reuses existing utilities
- [ ] No commented-out code
- [ ] Method length reasonable (single responsibility)
- [ ] Errors handled specifically — not caught generically and swallowed
- [ ] Logging at appropriate levels (debug for dev noise, info for business events, error for failures)
- [ ] No hardcoded environment-specific values (URLs, credentials)

---

### 5. Frontend-Specific (if applicable)

- [ ] Components are reasonably small and focused
- [ ] Props typed correctly — no `any`
- [ ] Side effects isolated (hooks/services, not inline in render)
- [ ] Accessibility attributes present where needed (`aria-*`, semantic HTML)
- [ ] No direct DOM manipulation bypassing the framework

---

## Review Output Format

```markdown
# Changeset Review: {CHANGESET-ID} — {title}

**Ticket:** {TICKET-ID}
**Date:** {YYYY-MM-DD}

## Summary

{One paragraph overall assessment}

## Findings

### Critical (must fix before merge)
- ...

### Major (should fix)
- ...

### Minor (suggestions / nits)
- ...

## Checklist Summary

{Checklist items checked/unchecked}
```

Output saved to: `docs/AI-analysis-plan-docs/{TICKET-ID}/REVIEW-{CHANGESET-ID}.md`
