---
name: write-tests-be
description: 'Generate backend unit and integration tests for a class or module. Reads context/best-practices/backend.md for framework and conventions, then produces a complete test file. Use when asked to "write tests", "generate tests", "add unit tests", "create test class" for backend code.'
---

# Write Backend Tests

## When to Use

Use this skill to generate unit and/or integration test files for existing backend classes.

**Input:** Target class path, file reference, or description (e.g. "write tests for MyService.java")  
**Output:** Test file placed alongside the source file in `src/test/`

---

## Prerequisites

Read before generating:
- `.github/context/best-practices/backend.md` — test framework, test class structure, naming convention, coverage targets, assertion library
- `.github/context/architecture.md` — module layout, dependencies, key concepts

---

## Steps

### Step 1 — Load Best Practices

Read `.github/context/best-practices/backend.md` in full. Extract:
- Test framework and imports (JUnit 5 / Mockito / AssertJ or equivalent)
- Test class structure (unit vs integration annotations)
- Test naming convention
- Assertion library preference
- Coverage targets

### Step 2 — Read the Target Class

Read the full content of the target class. Identify:
- Class name and package
- All public methods (these are the primary test targets)
- All constructor-injected dependencies (these become `@Mock` declarations)
- Return types and exception signatures
- Business logic branches (each branch needs a test case)

### Step 3 — Read Coupled Dependencies

For any dependency whose interface is non-trivial, read its class/interface to understand the contract being mocked.

### Step 4 — Identify Test Cases

For each public method, enumerate:

| Method | Scenario | Test type | Notes |
|--------|----------|-----------|-------|
| `process(input)` | Valid input → returns result | Unit | Happy path |
| `process(input)` | Null input → throws exception | Unit | Guard clause |
| `process(input)` | Dependency throws → error handled | Unit | Error path |
| `save(entity)` | Persists to DB and returns ID | Integration | Verify DB state |

Aim for: all branches covered, all exception paths covered, at least one integration test per service.

### Step 5 — Generate Test File

Follow the test class structure from `context/best-practices/backend.md` exactly.

**File location:** Replace `src/main/` with `src/test/` in the path, keeping the same package.  
**File name:** `{ClassName}Test.java` for unit tests, `{ClassName}IntegrationTest.java` for integration tests.

Apply these rules:
- One test class per file
- `@Mock` for every injected dependency
- `@InjectMocks` on the class under test (unit) or `@Autowired` (integration)
- Every test method follows the naming convention from `backend.md`
- Every test uses arrange / act / assert structure with blank-line separation
- Use the assertion library from `backend.md` (e.g. AssertJ, not JUnit assertions)
- No test that only checks framework behaviour (e.g. mock was called but nothing asserted about outcome)

### Step 6 — Validate

After generating, check:
- [ ] All public methods have at least one test
- [ ] All exception-throwing paths are tested
- [ ] Test names match the naming convention from `backend.md`
- [ ] Imports are complete and correct
- [ ] File is in the correct `src/test/` path with the same package as the source

If coverage targets from `backend.md` cannot be met by the generated tests, add a comment listing uncovered branches and why.
