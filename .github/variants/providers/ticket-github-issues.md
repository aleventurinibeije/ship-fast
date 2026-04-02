# Ticket Manager: GitHub Issues

> **STATUS: DRAFT** — This variant documents expected operations for GitHub Issues. MCP tool names use the GitHub MCP pattern. Adjust if using a different MCP server.

## MCP Loading

- **Load pattern:** `mcp_github`
- **Tool prefix:** `mcp_github___`

> **Deferred tools:** Call `tool_search_tool_regex` with the load pattern above **before** any tool call.

## Constants

| Constant | Value |
|----------|-------|
| Repository owner | `<<GITHUB_OWNER>>` |
| Repository name | `<<GITHUB_REPO>>` |
| Base URL | `https://github.com/<<GITHUB_OWNER>>/<<GITHUB_REPO>>` |
| Ticket pattern | `#\d+` |

## Operations

### fetch-ticket

Fetch an issue's full details.

- **Tool:** `mcp_github___get_issue`
- **Parameters:**
  - `owner`: Repository owner (from Constants)
  - `repo`: Repository name (from Constants)
  - `issue_number`: Numeric issue number
- **Response fields:**
  - `title` → issue title
  - `body` → full description (Markdown)
  - `state` → open / closed
  - `labels[]` → array of label objects
  - `assignee.login` → assignee username
  - `comments` → comment count

### update-ticket

Update an issue's fields.

- **Tool:** `mcp_github___update_issue`
- **Parameters:**
  - `owner`: Repository owner
  - `repo`: Repository name
  - `issue_number`: Numeric issue number
  - `body`: Updated description (Markdown)

## Output Format

GitHub Issues uses **GitHub-flavored Markdown**.

### Key GFM Rules

- Standard Markdown headings: `## Heading`
- Code blocks: triple backticks with language tag
- Tables: standard Markdown pipe tables
- Task lists: `- [ ] item` / `- [x] item`
- Inline code: backticks `` `code` ``
- Issue references: `#123` auto-links to issue
- User mentions: `@username`

### Validation Rules

- [ ] No Jira or Textile markup
- [ ] Pure GitHub-flavored Markdown
- [ ] Issue references use `#` prefix
- [ ] Task lists use `- [ ]` format
