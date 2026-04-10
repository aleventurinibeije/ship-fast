# <<PROJECT_NAME>> Workspace

**<<PROJECT_NAME>>** — short one-line description of your project.

---

## Agent Registry

| # | Task | Agent |
|---|------|-------|
| 1 | Configure .github/context/ for a new project | `01-bootstrap` |
| 2 | Generate PRD + CONTEXT-SNAPSHOT from raw docs in docs/ | `02-context-builder` |
| 3 | Create or update the PRD from analysis docs and user stories | `03-prd-writer` |
| 4 | Create tickets from PRD and sync references back | `04-prd-tickets` |
| 5 | Convert design files to component specifications | `05-design` |
| 6 | Create or update a technical analysis document for a ticket | `06-analysis` |
| 7 | Write a ticket description in the configured tracker's format | `07-ticket-description` |
| 8 | Implement a ticket, fix a bug, write tests | `08-dev` |
| 9 | Review a changeset (MR/PR) by URL | `09-review` |
| 10 | Write a changeset (MR/PR) description for a ticket | `10-changeset-description` |

---

## MCP Providers — Dynamic Loading

MCP tools for external integrations are **deferred** — they are NOT available until explicitly loaded via `tool_search_tool_regex`.

**Provider configs live in `.github/context/providers/`.** Each config file documents:
- The `tool_search_tool_regex` load pattern
- Tool names and parameter mappings for each operation
- Output format rules
- Project-specific constants

**How skills load providers:**

1. Read the provider config file (e.g. `context/providers/ticket-manager.md`)
2. Extract the load pattern from the `## MCP Loading` section
3. Call `tool_search_tool_regex` with that pattern — **wait for results before proceeding**
4. Call operations using the tool names and parameters from the provider's `## Operations` section

**Provider slots:**

| Slot | Config file | Purpose |
|------|------------|---------|
| Ticket manager | `context/providers/ticket-manager.md` | Issue tracker (Jira, Redmine, GitHub Issues, etc.) |
| Code review | `context/providers/code-review.md` | MR/PR platform (GitLab, GitHub, Bitbucket, etc.) |
| Design tool | `context/providers/design-tool.md` | Design tool (Figma, Sketch, etc.) — optional |

See `context/providers/LOADING-PROTOCOL.md` for the shared loading steps used by all skills.
