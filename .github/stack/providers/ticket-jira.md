# Ticket Manager: Jira (Atlassian MCP)

## MCP Loading

- **Load pattern:** `mcp_io_modelconte_atlassian`
- **Tool prefix:** `mcp_io_modelconte_atlassian-mcp___`

> **Deferred tools:** Call `tool_search_tool_regex` with the load pattern above **before** any tool call. Tools are NOT available until loaded.

## Constants

| Constant | Value |
|----------|-------|
| Project key | `<<JIRA_PROJECT_KEY>>` |
| Base URL | <!-- e.g. https://jira.example.com --> |
| Ticket pattern | `<<JIRA_PROJECT_KEY>>-\d+` |

## Operations

### fetch-ticket

Fetch a ticket's full details.

- **Tool:** `mcp_io_modelconte_atlassian-mcp___jira_get_issue`
- **Parameters:**
  - `issue_key`: The ticket ID (e.g. `MYAPP-123`)
- **Response fields:**
  - `fields.summary` → ticket title
  - `fields.description` → full description
  - `fields.issuetype.name` → Story / Bug / Task
  - `fields.status.name` → current status
  - `fields.assignee.displayName` → assignee
  - `fields.comment.comments[-3:]` → last 3 comments
  - `fields.issuelinks` → linked issues

### update-ticket

Update a ticket's fields (e.g. push a description).

- **Tool:** `mcp_io_modelconte_atlassian-mcp___jira_update_issue`
- **Parameters:**
  - `issue_key`: The ticket ID
  - `fields`: JSON object of fields to update (e.g. `{ "description": "..." }`)

### search-tickets

Search for tickets using JQL.

- **Tool:** `mcp_io_modelconte_atlassian-mcp___jira_search`
- **Parameters:**
  - `jql`: JQL query string (e.g. `project = MYAPP AND status = "In Progress"`)
  - `max_results`: Max results to return (default: 10)

## Output Format

Jira uses **Jira wiki markup** — NOT Markdown. All description-writing skills must use these rules.

### Text Formatting

| Element | Syntax |
|---|---|
| Heading 1 | `h1. Title` |
| Heading 2 | `h2. Title` |
| Heading 3 | `h3. Title` |
| Bold | `*text*` |
| Italic | `_text_` |
| Monospace / inline code | `{{text}}` |
| Horizontal rule | `----` |
| Note panel | `{note}text{note}` |
| Warning panel | `{warning}text{warning}` |

### CRITICAL: Curly-brace escaping

Jira interprets `{…}` as a macro invocation.

**Rule:** Inside `{{…}}` monospace, any literal `{` or `}` MUST be escaped with a backslash:
- `{` → `\{`
- `}` → `\}`

**Examples:**
- `@Profile({"local","test"})` → `{{@Profile(\{"local","test"\})}}`
- `Map<String, Object>` — angle brackets are safe, no escaping needed

### Code Blocks

```
{code:java}
// Java code here
{code}

{code:yaml}
# YAML here
{code}

{code:typescript}
// TypeScript here
{code}
```

**Never** use triple-backtick fences — they are Markdown, not Jira.

### Lists

Unordered: `* item` (do NOT use `- item`)
Ordered: `# item` (do NOT use `1. item`)
Nested: `** sub-item`, `## sub-numbered`

**CRITICAL: `#` at line start means ordered list item. Never use `# Heading` — use `h1.` / `h2.` instead.**

### Tables

Header row: `||Column A||Column B||`
Data rows: `|cell value|cell value|` — **no spaces** between pipe and cell content.

### Checkboxes

Unchecked: `* ( ) task text`

### Links

`[display text|https://example.com]`

Ticket self-link: `[{TICKET-ID}|{BASE_URL}/browse/{TICKET-ID}]`

### Validation Rules

Before finishing any Jira-format output, verify:

- [ ] No Markdown syntax anywhere (no `##`, no `-`, no backticks)
- [ ] Inner `{`/`}` inside `{{…}}` escaped with `\`
- [ ] Table data rows have no spaces at the pipe boundaries
- [ ] Code blocks use `{code:lang}…{code}`
- [ ] `#` only used for ordered list items, never headings
- [ ] Horizontal rules are `----`
- [ ] Checkboxes use `* ( ) text` format
