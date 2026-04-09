# stack/ — Portable Stack & Provider Templates

This folder contains **setup-time templates** used by the `bootstrap` skill to configure `.github/context/` for a new project.

> **Runtime note:** These files are NOT loaded by any agent or skill at runtime. They are source templates only — content is copied into `context/` during bootstrap.

---

## Stack variants

| File | Stack | Use for |
|------|-------|---------|
| `be-node-platformatic.md` | Node.js + TypeScript + Platformatic DB | Platformatic monorepo projects |
| `be-node-generic.md` | Node.js + TypeScript (Fastify / Express / NestJS) | Generic Node.js backends |
| `be-java-spring.md` | Java 17 + Spring Boot | Java Spring backends |
| `fe-react-ts.md` | React + TypeScript (Vite / Next.js) | React frontend projects |
| `fe-vue.md` | Vue 3 + TypeScript | Vue frontend projects |

Bootstrap detection heuristics:

| Signal | Stack selected |
|--------|---------------|
| `package.json` dep: `@platformatic/*` | `be-node-platformatic.md` |
| `package.json` dep: `fastify` / `express` / `@nestjs/*` | `be-node-generic.md` |
| `pom.xml` or `build.gradle` present | `be-java-spring.md` |
| `package.json` dep: `react` or `next` | `fe-react-ts.md` |
| `package.json` dep: `vue` | `fe-vue.md` |

**Monorepo:** Bootstrap checks each sub-package independently and copies the appropriate stack file per package into `context/best-practices/<package-name>.md`.

---

## Provider templates (`providers/`)

| File | Provider |
|------|---------|
| `providers/ticket-redmine.md` | Redmine REST API |
| `providers/ticket-jira.md` | Jira (Atlassian MCP) |
| `providers/ticket-github-issues.md` | GitHub Issues MCP |
| `providers/review-github.md` | GitHub Pull Requests MCP |
| `providers/review-gitlab.md` | GitLab Merge Requests MCP |
| `providers/design-figma.md` | Figma MCP |

Bootstrap copies the selected provider template to `context/providers/ticket-manager.md`, `context/providers/code-review.md`, and `context/providers/design-tool.md`, then injects project-specific constants (owner, repo, project ID, base URL, API key env var).

---

## How to port `.github/` to a new project

1. Copy the entire `.github/` folder into the new project root
2. Delete `context/` files that are project-specific (PRD.md, CONTEXT-SNAPSHOT.md, architecture.md, integrations.md, best-practices/, providers/ — keep LOADING-PROTOCOL.md)
3. Run `@bootstrap` — the skill will detect the stack, ask for provider constants, and regenerate `context/`
4. If the project has existing documentation in `docs/`, run `@context-builder` — it will generate `PRD.md` and `CONTEXT-SNAPSHOT.md` from those docs
