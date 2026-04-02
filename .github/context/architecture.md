<!-- FILL IN: System Architecture & Tech Stack
     AGENTS USE THIS FILE FOR:
     - analyze-ticket: understands module layout before reading code
     - dev: knows which files to touch, which patterns to follow
     - mr-review: validates changes fit the established architecture
     - write-tests-be / write-tests-fe: detects tech stack to pick test patterns

     WHAT TO INCLUDE:
     - Tech stack (languages, frameworks, versions)
     - Module/service layout with directory map
     - Key concepts and domain model
     - Data flow / pipeline description
     - State machine (if applicable)
     - System boundaries and integration points
     - Architectural invariants that must never be violated
-->

# <<PROJECT_NAME>> — Architecture

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend language | <!-- e.g. Java 17 --> |
| Backend framework | <!-- e.g. Spring Boot 3.x --> |
| Frontend language | <!-- e.g. TypeScript --> |
| Frontend framework | <!-- e.g. React 18 --> |
| Database | <!-- e.g. PostgreSQL 15 --> |
| Schema migrations | <!-- e.g. Liquibase --> |
| Build tool | <!-- e.g. Maven / npm --> |
| Test framework (BE) | <!-- e.g. JUnit 5 + Mockito --> |
| Test framework (FE) | <!-- e.g. Jest + React Testing Library --> |

## Module Layout

```
src/
  <!-- Paste your directory structure here -->
```

## Key Concepts

<!-- Domain entities, core abstractions, vocabulary used across the codebase -->

## Data Flow

<!-- Describe how data moves through the system end-to-end -->

## State Machine (if applicable)

<!-- List states, transitions, and terminal states -->

## System Boundaries

<!-- What external systems does this project integrate with? -->

## Architectural Invariants

<!-- Rules that must never be violated — enforced by agents during review -->
<!-- Example: "All DB writes go through the Repository layer — never raw SQL in services" -->
