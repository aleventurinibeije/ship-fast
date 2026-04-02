---
name: run-test-coverage
description: 'Run unit and integration tests, analyze coverage, and report results. Reads test commands from context/integrations.md. Use when asked to "run tests", "check coverage", "test coverage report".'
---

# Run Test Coverage

## When to Use

Use this skill to run tests and analyze code coverage.

**Input:** Optional scope — "backend", "frontend", specific test class/file name, or nothing (runs all)  
**Output:** Test results summary + coverage report

---

## Prerequisites

Read before starting:
- `.github/context/integrations.md` — test commands for BE and FE
- `.github/context/best-practices/backend.md` — BE coverage targets (if running BE tests)
- `.github/context/best-practices/frontend.md` — FE coverage targets (if running FE tests)

---

## Steps

### Step 1 — Determine Scope

If the user specified a scope, use it. Otherwise ask:
> "Should I run backend tests, frontend tests, or both?"

Read test commands from `.github/context/integrations.md` → `## Test Commands`.

### Step 2 — Run Tests

**Backend:**
```bash
# Read the exact command from context/integrations.md → Test Commands → Backend
```

**Frontend:**
```bash
# Read the exact command from context/integrations.md → Test Commands → Frontend
```

**Specific test class/file:**  
Read the specific test command from `context/integrations.md` and substitute the class/file name.

Capture:
- Total tests run
- Passed / Failed / Skipped
- Failure messages (if any)

### Step 3 — Generate Coverage Report

**Backend:**
```bash
# Read the coverage report command from context/integrations.md → Test Commands → Backend
```

**Frontend:**
```bash
# Coverage is typically included in the test run — check context/integrations.md
```

### Step 4 — Analyze Results

Read the coverage report output. Compare against targets from:
- `context/best-practices/backend.md` → `### Coverage Targets`
- `context/best-practices/frontend.md` → `### Coverage Targets`

Identify packages/files below target thresholds.

### Step 5 — Report

```markdown
## Test Results

| Metric | Value |
|--------|-------|
| Total | {n} |
| Passed | {n} |
| Failed | {n} |
| Skipped | {n} |

### Failures (if any)

- `{TestClass}#{method}` — {error message}

## Coverage

| Package / Module | Line % | Branch % | Status |
|------------------|--------|----------|--------|
| com.example.service | 85% | 72% | ✅ |
| com.example.client  | 61% | 45% | ❌ below target |

**Targets:** Line ≥ {x}%, Branch ≥ {y}% (from context/best-practices/backend.md or frontend.md)

### Coverage Gaps

{List files/classes below target with suggested test additions}
```
