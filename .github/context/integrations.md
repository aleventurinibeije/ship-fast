<!-- FILL IN: External Integrations & Project Constants
     AGENTS USE THIS FILE FOR:
     - run-test-coverage: reads test commands
     - write-tests-be / write-tests-fe: reads test runner commands
     - dev: reads external API endpoints

     MCP INTEGRATIONS (ticket manager, code review, design tool):
     These are configured separately in context/providers/.
     See context/providers/README.md for setup instructions.

     WHAT TO INCLUDE HERE:
     - Test commands for BE and FE
     - External API endpoints (dev/prod)
     - Environment variable names (not values)
-->

# <<PROJECT_NAME>> — Integrations & Constants

> **MCP provider configs** (ticket manager, code review, design tool) live in `.github/context/providers/`. This file covers non-MCP integrations: test commands, external APIs, and environment variables.

## Test Commands

### Backend

```bash
# Run all BE tests
<!-- e.g. cd backend && mvn clean test -->

# Run specific test class
<!-- e.g. mvn test -Dtest=MyServiceTest -->

# Generate coverage report
<!-- e.g. mvn jacoco:report -->
```

### Frontend

```bash
# Run all FE tests
<!-- e.g. cd frontend && npm test -->

# Run with coverage
<!-- e.g. npm test -- --coverage -->

# Run specific file
<!-- e.g. npm test -- src/components/MyComponent.test.tsx -->
```

## External APIs

| Service | Dev URL | Prod URL | Auth |
|---------|---------|----------|------|
| <!-- Name --> | <!-- URL --> | <!-- URL --> | <!-- e.g. API key via Bearer token --> |

## Environment Variables

| Variable | Purpose |
|----------|---------|
| <!-- VAR_NAME --> | <!-- what it controls --> |
