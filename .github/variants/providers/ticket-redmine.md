# Ticket Manager: Redmine

> **STATUS: DRAFT** — This variant documents the expected operations and format. MCP tool names are placeholders (`<<TODO>>`). Fill them in when a Redmine MCP server is available.

## MCP Loading

- **Load pattern:** `<<TODO_REDMINE_MCP_PATTERN>>`
- **Tool prefix:** `<<TODO_REDMINE_MCP_PREFIX>>`

> **Deferred tools:** Call `tool_search_tool_regex` with the load pattern above **before** any tool call.

## Constants

| Constant | Value |
|----------|-------|
| Project identifier | `<<REDMINE_PROJECT_ID>>` |
| Base URL | <!-- e.g. https://redmine.example.com --> |
| Ticket pattern | `<<REDMINE_PROJECT_ID>>-\d+` or `#\d+` |

## Operations

### fetch-ticket

Fetch an issue's full details.

- **Tool:** `<<TODO>>`
- **Parameters:**
  - `issue_id`: Numeric issue ID
- **Response fields:**
  - `issue.subject` → ticket title
  - `issue.description` → full description
  - `issue.tracker.name` → Bug / Feature / Task
  - `issue.status.name` → current status
  - `issue.assigned_to.name` → assignee

### update-ticket

Update an issue's fields.

- **Tool:** `<<TODO>>`
- **Parameters:**
  - `issue_id`: Numeric issue ID
  - `notes`: Comment text (optional)
  - `fields`: Fields to update

## Output Format

Redmine uses **Textile markup**.

### Key Textile Rules

| Element | Syntax |
|---|---|
| Heading 1 | `h1. Title` |
| Heading 2 | `h2. Title` |
| Bold | `*text*` |
| Italic | `_text_` |
| Inline code | `@code@` |
| Code block | `<pre><code class="java">...</code></pre>` |
| Unordered list | `* item` |
| Ordered list | `# item` |
| Table header | `|_. Col A |_. Col B |` |
| Table row | `| cell | cell |` |
| Link | `"text":url` |

### Validation Rules

- [ ] No Markdown syntax (no `##`, no backticks)
- [ ] Inline code uses `@…@` not backticks
- [ ] Code blocks use `<pre>` tags
- [ ] Tables use `|_. ` for headers
- [ ] Links use `"text":url` format
