---
name: bootstrap
description: 'Configure .github/context/ for a new project. Detects the tech stack, asks for provider constants (GitHub, Redmine/Jira, optional Figma), copies the right stack template from stack/, and scaffolds all context/ files. Use when asked to "bootstrap", "setup project", "configure for new project", "port to new project", "adapt .github for this project".'
---

# Bootstrap

## When to Use

Use this skill when `.github/` has been copied into a new project and needs to be configured for it. It:

1. Detects the project's tech stack (per-package in monorepos)
2. Detects whether the project is **existing** (has real source code) or **greenfield**
3. Collects provider constants from the user (GitHub repo, ticket tracker, optional design tool)
4. Copies the right stack template(s) from `stack/` into `context/best-practices/`
5. **For existing projects**: introspects source code to extract project-specific conventions and appends them to the best-practices files
6. Overwrites `context/providers/` files with provider templates + injected constants
7. **For existing projects**: generates a real `context/architecture.md` from the codebase instead of a placeholder
8. Scaffolds empty placeholder files for any missing context files
9. Offers to run `build-context` if the project already has docs

> **No provider needed before bootstrap runs.** Provider files are the OUTPUT of this skill, not the input.

---

## Prerequisites

Read before starting:
- `stack/README.md` — lists available stack files and detection heuristics
- `stack/providers/` — list all available provider templates

---

## Steps

### Step 1 — Detect Project Structure

1. Check if the project is a monorepo:
   - Look for `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `packages/`, or multiple `package.json` at depth 2
   - If root has `watt.json` alongside `pnpm-workspace.yaml` → Platformatic monorepo
2. **For each package** (or root if not a monorepo), detect the stack:

| Check | Stack |
|-------|-------|
| `package.json` has `@platformatic/db` or `@platformatic/service` | `be-node-platformatic` |
| `package.json` has `fastify` / `express` / `@nestjs/core` (no Platformatic) | `be-node-generic` |
| `pom.xml` or `build.gradle` present | `be-java-spring` |
| `package.json` has `react` or `next` | `fe-react-ts` |
| `package.json` has `vue` | `fe-vue` |
| `vite.config.*` present + `package.json` has `react` | `fe-react-ts` |
| None of the above | unknown — ask user |

3. Collect results into a list: `[{ package: "web/platformatic-db", stack: "be-node-platformatic" }, ...]`

---

### Step 2 — Confirm Stack with User

Present findings and ask for confirmation using `vscode/askQuestions`:

```
Detected the following stacks:
- web/platformatic-db → be-node-platformatic
- web/platformatic-gateway → be-node-generic

Is this correct? If any package is wrong, specify the correct stack from: be-node-platformatic, be-node-generic, be-java-spring, fe-react-ts, fe-vue
```

Accept corrections before proceeding.

---

### Step 2b — Detect Project Maturity

For each confirmed package, count the number of non-trivial source files:

- **Node.js / Platformatic / generic BE**: look in `routes/`, `plugins/`, `schemas/`, `test/`, `src/`, `migrations/`
- **Java Spring**: look in `src/main/java/`, `src/test/java/`
- **React / Vue FE**: look in `src/components/`, `src/views/`, `src/pages/`, `src/hooks/`, `src/stores/`

Count files that are **not** index stubs (i.e. more than ~5 lines of substantive code).

| Source files found | Classification |
|--------------------|----------------|
| 0–4 | **Greenfield** — use generic stack template only |
| 5 or more | **Existing project** — enable code extraction (Steps 4b and 6b) |

Store the result per package: `[{ package: "web/platformatic-db", stack: "be-node-platformatic", maturity: "existing" }, ...]`

> If extraction is enabled for at least one package, ask the user once:
> ```
> Found an existing codebase in {packages}. I can introspect your source files to extract 
> project-specific conventions and generate a real architecture.md. This may take a moment.
> Proceed with code extraction? (Y/N)
> ```
> If the user declines, treat all packages as greenfield for the remaining steps.

---

### Step 3 — Collect Provider Constants

Ask the user for the following constants via `vscode/askQuestions` (all required unless marked optional):

**Code review (GitHub):**
- GitHub owner / organization (e.g. `wrap-repo`)
- GitHub repository name (e.g. `LAB-OmniseeAiBE`)

**Ticket manager — ask which tracker:**
- Options: Redmine, Jira, GitHub Issues
- If **Redmine**:
  - Base URL (e.g. `https://pm.example.com`)
  - Project identifier (e.g. `my-project`)
  - API key env var name (e.g. `REDMINE_API_KEY`)
  - Tracker ID for **parent/module tickets** (e.g. Epic, Feature — find the ID at `{base-url}/trackers`)
  - Tracker ID for **child/activity tickets** (e.g. Task — find the ID at `{base-url}/trackers`)
  - Status ID for **Open** (e.g. `1` — find at `{base-url}/issue_statuses.json`)
  - Status IDs for workflow states *(optional — leave blank to skip)*: In Analysis, In Development, In Review
- If **Jira**: Jira base URL, project key (e.g. `MYAPP`), Atlassian MCP already configured? (Y/N)
- If **GitHub Issues**: same owner/repo as code review (confirm)

> **Tip for finding Redmine IDs:** Open `{base-url}/trackers` (requires admin) for tracker IDs and `{base-url}/issue_statuses.json` for status IDs. Alternatively, fetch any issue via the API — the `tracker.id` and `status.id` fields are included in the response.

**Design tool (optional):**
- Figma enabled? (Y/N)
- If yes: Figma token env var name (e.g. `FIGMA_TOKEN`)

---

### Step 4 — Copy Stack Files

For each detected (and confirmed) package:

1. Read `stack/{stack-name}.md`
2. If single-package project (or all packages share the same stack): copy to `context/best-practices/backend.md` (or `frontend.md`)
3. If monorepo with multiple packages: copy to `context/best-practices/{package-short-name}.md`
   - e.g. `web/platformatic-db` → `context/best-practices/platformatic-db.md`
   - e.g. `web/platformatic-gateway` → `context/best-practices/platformatic-gateway.md`
4. Add a comment header at the top of the copied file:
   ```
   <!-- Copied from stack/{stack-name}.md by bootstrap on YYYY-MM-DD. Edit this file to match your project's specifics. -->
   ```

---

### Step 4b — Code Extraction (existing projects only)

Skip this step entirely if the package is **greenfield** or the user declined extraction.

For each **existing** package, run a targeted codebase exploration using `semantic_search`, `grep_search`, and `read_file`. Collect findings across the following dimensions:

#### Routes
- Naming conventions (file-per-resource, flat, nested?)
- How schemas are attached (`schema: { body, querystring, response }`)
- Auth/guard decoration patterns (`preHandler`, decorators used)
- Error response shape and typed error classes used
- Whether routes use generics (`fastify.get<{ Params: ... }>`) or plain handlers
- Common HTTP status codes returned

#### Plugins
- How plugins are registered and grouped (auto-load, manual import, barrel files?)
- What decorators are added to `app`/`fastify`
- Lifecycle hooks used (`onRequest`, `preHandler`, `onSend`, etc.)
- Shared utilities / helpers pattern

#### Schemas / Types
- Schema definition style (TypeBox, Zod, JSON Schema literals, etc.)
- Co-location rule: in route file, in dedicated `schemas/` dir, or shared barrel?
- How types are derived from schemas (`Static<typeof ...>`)
- Naming conventions for schema constants

#### Tests
- Test framework and runner (tap, jest, vitest, mocha?)
- File naming convention (`*.test.ts`, `*.spec.ts`, `test/*.ts`?)
- How the app is bootstrapped in tests (`buildServer()`, `fastify()`, inject?)
- Mocking strategy (sinon, vi.mock, nock, custom fixtures?)
- Whether tests use a real DB or mocks

#### Migrations & Config
- Migration file naming scheme and observed entity names
- Env var patterns (from `watt.json`, `.env.sample`, or `process.env` references in code)
- External integrations referenced (auth providers, external APIs, queues, etc.)

#### Synthesis

After collecting findings, produce a structured summary:

```markdown
## Project-Specific Conventions
<!-- Auto-extracted by bootstrap on YYYY-MM-DD. Review and edit as needed. -->

### Observed Patterns

| Area | Convention observed |
|------|---------------------|
| Routes | e.g. One file per resource under routes/, TypeBox schema in same file |
| Plugins | e.g. Auto-loaded from plugins/; app decorated with `currentUser` |
| Schemas | e.g. TypeBox + Static<>; constants named `{Entity}Schema` |
| Tests | e.g. node:test + tap-style assertions; app bootstrapped via buildApp() helper |
| Migrations | e.g. NNN.do.sql / NNN.undo.sql; entities: resident, building, worker |
| Env vars | e.g. Injected via watt.json; accessed via process.env.LOGTO_ENDPOINT |

### Deviations from Generic Template
<!-- List any places where the actual code differs from the generic template rules -->
- e.g. "Raw SQL found in routes/workers.ts:42 — deviates from 'no ORM' rule"
- e.g. "Uses dotenv — deviates from 'no dotenv' convention"

### Key Entity Names
<!-- Domain entities found in migrations / schemas -->
- e.g. resident, building, worker, package, gate
```

**Append** this section to the end of the `context/best-practices/{package}.md` file written in Step 4.  
Do NOT modify the generic template content above it.  
If the existing best-practices file already contains a `## Project-Specific Conventions` section, replace only that section (do not duplicate).

---

### Step 5 — Write Provider Config Files

**code-review.md:**
1. Read `stack/providers/review-github.md` (or `review-gitlab.md` if user chose GitLab)
2. Replace all `{{OWNER}}` placeholders with the collected owner value
3. Replace all `{{REPO}}` placeholders with the collected repo name value
4. Write to `context/providers/code-review.md`

**ticket-manager.md:**
1. Read the matching template from `stack/providers/`:
   - Redmine → `ticket-redmine.md`
   - Jira → `ticket-jira.md`
   - GitHub Issues → `ticket-github-issues.md`
2. Replace connection placeholders with collected values:
   - `<<REDMINE_BASE_URL>>` → base URL
   - `<<REDMINE_PROJECT_ID>>` → project identifier
   - `<<REDMINE_API_KEY>>` → API key env var name
   - `<<JIRA_PROJECT_KEY>>` / `<<JIRA_BASE_URL>>` → Jira equivalents
   - `<<GITHUB_OWNER>>` / `<<GITHUB_REPO>>` → GitHub Issues equivalents
3. Replace tracker/status ID placeholders with collected values:
   - `<<TRACKER_ID_PARENT>>` → parent ticket tracker ID (or leave placeholder if not collected)
   - `<<TRACKER_ID_CHILD>>` → child ticket tracker ID
   - `<<STATUS_ID_OPEN>>` → open status ID
   - `<<STATUS_ID_IN_ANALYSIS>>` → in-analysis status ID (replace with empty string if user skipped)
   - `<<STATUS_ID_IN_DEV>>` → in-development status ID (replace with empty string if user skipped)
   - `<<STATUS_ID_IN_REVIEW>>` → in-review status ID (replace with empty string if user skipped)
4. Write to `context/providers/ticket-manager.md`

**design-tool.md:**
- If Figma enabled: read `stack/providers/design-figma.md`, replace `{{FIGMA_TOKEN_ENV}}`, write to `context/providers/design-tool.md`
- If not enabled: write a N/A banner to `context/providers/design-tool.md`:
  ```markdown
  <!-- N/A: No design tool configured for this project. Skills must skip all design MCP steps. -->
  ```

---

### Step 6 — Scaffold Missing Context Files

For each of these files, if it does NOT already exist, create it with a placeholder header:

**`context/CONTEXT-SNAPSHOT.md`**:
```markdown
> ⚠️ PLACEHOLDER — Run `@context-builder` to generate this file from your docs/, or fill it in manually.
> Last updated: YYYY-MM-DD (bootstrap scaffolded)

# Context Snapshot — {PROJECT_NAME}

## Tech Stack
| Layer | Technology |
|-------|-----------|
| ⚠️ | Fill in after running bootstrap |

## Key File Locations
⚠️ Fill in manually.

## Domain Model
⚠️ Run `@context-builder` or fill in manually.

## Auth Layers
⚠️ Fill in manually.

## Mandatory Conventions
⚠️ Fill in manually or run `@context-builder`.

## Testing
⚠️ Fill in manually (test commands, coverage targets).
```

**`context/PRD.md`**:
```markdown
> ⚠️ PLACEHOLDER — Run `@context-builder` to generate this file from your docs/, or fill it in manually following .github/instructions/prd.instructions.md.
```

**`context/architecture.md`**:
```markdown
> ⚠️ PLACEHOLDER — Fill in manually: service diagram, module layout tree, Architectural Invariants.
```
> **Skip this placeholder** if Step 6b already wrote a real `context/architecture.md`.

**`context/integrations.md`**:
```markdown
> ⚠️ PLACEHOLDER — Fill in manually: test commands (from package.json / pom.xml), external APIs, env var table.
```

If a file already exists (non-empty), skip it — do NOT overwrite.

---

### Step 6b — Architecture Extraction (existing projects only)

Skip this step if all packages are **greenfield**, the user declined extraction, or `context/architecture.md` already exists with real content (i.e. does not contain the `⚠️ PLACEHOLDER` string).

Explore the codebase to produce a real `context/architecture.md`. Use the findings already gathered in Step 4b, plus targeted reads of top-level config files.

#### Service Layout
- Read root `watt.json` (if present) to find registered services and their paths
- Map inter-service relationships (gateway → db, any sidecar services)
- Note the runtime orchestrator (WattPM, Docker Compose, etc.)

#### Module Layout Tree
- List the directory tree of each package's source root to depth 2
- Annotate each top-level directory with its role (e.g. `routes/` — HTTP handlers, `plugins/` — Fastify plugins/decorators, `migrations/` — DB schema, `test/` — integration tests)

#### Domain Model
- List all entities discovered in Step 4b (from migrations, schema constants, entity API usages)
- For each entity, note key fields if visible in migration SQL or TypeBox schemas

#### Auth Layers
- Identify auth mechanisms (JWT middleware, Logto, API key, session, etc.) from plugins and route decorators
- Note which routes are public vs protected

#### Architectural Invariants
- Summarise the hard rules observed (extracted from constraints visible in code and from the generic best-practices template)
- Flag anything that looks like a deliberate architectural boundary (e.g. "DB service never imported directly; always accessed via gateway")

#### Output format

Write `context/architecture.md` using this structure:

```markdown
<!-- Auto-generated by bootstrap on YYYY-MM-DD from codebase introspection. Review and update as the project evolves. -->

# Architecture — {PROJECT_NAME}

## Service Diagram

```
{plain-text diagram, e.g.:}
[Client] → [platformatic-gateway :3000] → [platformatic-db :5042]
                                           └── [PostgreSQL]
```

## Module Layout

### {package-name} (`{path}`)
```
routes/         HTTP route handlers (one file per resource)
plugins/        Fastify plugins, decorators, lifecycle hooks  
schemas/        (if present) Shared TypeBox schemas
migrations/     Raw SQL migration files (NNN.do.sql / NNN.undo.sql)
test/           Integration tests
types/          Shared TypeScript types
```

## Domain Model

| Entity | Key fields | Notes |
|--------|-----------|-------|
| ...    | ...       | ...   |

## Auth Layers

| Layer | Mechanism | Scope |
|-------|-----------|-------|
| ...   | ...       | ...   |

## Architectural Invariants

1. ...
2. ...
```

If a section cannot be determined from the code, write `> ⚠️ Could not be inferred — fill in manually.` for that section only.

---

### Step 7 — Offer to Run build-context

After scaffolding:

1. List all `.md` files found in `docs/` (if the folder exists).
2. If files are found, ask the user:
   ```
   Found {N} document(s) in docs/. Would you like to run `@context-builder` now to generate 
   context/PRD.md and context/CONTEXT-SNAPSHOT.md from them?
   ```
3. If yes: invoke the `build-context` skill.
4. If no (or `docs/` is empty): print a summary of what bootstrap created and list the manual fill-in items from the scaffolded placeholder files.
