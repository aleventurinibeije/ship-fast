---
name: write-tests-fe
description: 'Generate frontend component, hook, and unit tests. Reads context/best-practices/frontend.md for framework and conventions, then produces a complete test file. Use when asked to "write tests", "generate tests", "add tests", "create test file" for frontend components, hooks, or utilities.'
---

# Write Frontend Tests

## When to Use

Use this skill to generate test files for existing frontend components, hooks, or utility functions.

**Input:** Target file path or description (e.g. "write tests for MyComponent.tsx", "add tests for useMyHook.ts")  
**Output:** Test file placed alongside the source file

---

## Prerequisites

Read before generating:
- `.github/context/best-practices/frontend.md` — test framework, component test structure, hook test structure, naming convention, query selector priority, coverage targets
- `.github/context/architecture.md` — frontend module layout, state management approach

---

## Steps

### Step 1 — Load Best Practices

Read `.github/context/best-practices/frontend.md` in full. Extract:
- Test framework and imports (Jest, Vitest, React Testing Library, Vue Test Utils, etc.)
- Component test structure (render, interact, assert pattern)
- Hook test structure (renderHook / composable pattern)
- Query selector priority (which `getBy*` to prefer)
- Test naming convention
- Coverage targets

### Step 2 — Read the Target File

Read the full content of the target file. Identify:
- Component name / hook name / function name
- Props interface (for components)
- Return value shape (for hooks)
- User interactions available (button clicks, input changes, etc.)
- Conditional rendering branches (each branch needs a test case)
- Side effects (API calls, context updates, event emissions)

### Step 3 — Read Context Providers / Dependencies

If the component/hook consumes a Context, external service, or store, read those to understand the contract that needs to be mocked.

### Step 4 — Identify Test Cases

For components, enumerate:

| Scenario | What to assert |
|----------|---------------|
| Default render | Required elements visible |
| Conditional: prop X = true | Element Y shown |
| Conditional: prop X = false | Element Y hidden |
| Interaction: button clicked | Callback called / state changes |
| Loading state | Button disabled, spinner visible |
| Error state | Error message displayed |

For hooks:
- Initial state
- Each action's effect on state
- Async actions (pending / resolved / rejected)

### Step 5 — Generate Test File

Follow the test structure from `context/best-practices/frontend.md` exactly.

**File location:** Same directory as source file.  
**File name:** `{FileName}.test.tsx` / `{FileName}.test.ts` matching the source extension.

Apply these rules:
- `describe('{ComponentName}')` wrapping all tests
- Each test starts with a `defaultProps` or minimal valid setup
- `beforeEach(() => jest.clearAllMocks())` (or `vi.clearAllMocks()`)
- Use query selectors in the priority order from `frontend.md`
- Use `userEvent` (RTL) or `.trigger()` (VTU) for interactions — not `fireEvent` unless explicitly preferred
- Every test follows arrange / act / assert with blank-line separation
- No snapshot tests unless the project explicitly uses them (see `frontend.md`)
- Accessibility: verify via roles where possible, not CSS classes

### Step 6 — Validate

After generating, check:
- [ ] All rendering branches have a test
- [ ] All user interactions have a test
- [ ] All async states (loading, error, success) tested
- [ ] Query selectors use the priority from `frontend.md`
- [ ] Test names match the naming convention
- [ ] Imports are complete and correct
- [ ] File is in the correct directory

If coverage targets from `frontend.md` cannot be met, add a comment listing uncovered branches and why.
