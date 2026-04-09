# <<PROJECT_NAME>> Workspace

**<<PROJECT_NAME>>** — short one-line description of your project.

---

## Agent Registry

| Task | Agent |
|------|-------|
| Implement a ticket, fix a bug, write tests | `dev` |
| Create or update a technical analysis document for a ticket | `analysis` |
| Convert design files to component specifications | `design` |
| Review a changeset (MR/PR) by URL | `review` |
| Write a changeset (MR/PR) description for a ticket | `changeset-description` |
| Write a ticket description in the configured tracker's format | `ticket-description` |
| Create tickets from PRD and sync references back | `prd-tickets` |
| Configure .github/context/ for a new project | `bootstrap` |
| Generate PRD + CONTEXT-SNAPSHOT from raw docs in docs/ | `context-builder` |
| Create or update the PRD from analysis docs and user stories | `prd-writer` |

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
