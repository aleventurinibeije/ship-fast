# Code Review: GitHub PR

> **STATUS: DRAFT** — This variant documents expected operations for GitHub Pull Requests. MCP tool names use the GitHub MCP pattern. Adjust if using a different MCP server.

## MCP Loading

- **Load pattern:** `mcp_io_github_git`
- **Tool prefix:** `mcp_io_github_git_`

> **Deferred tools:** Call `tool_search_tool_regex` with the load pattern above **before** any tool call.

## Constants

| Constant | Value |
|----------|-------|
| Repository owner | `<<GITHUB_OWNER>>` |
| Repository name | `<<GITHUB_REPO>>` |
| Base URL | `https://github.com/<<GITHUB_OWNER>>/<<GITHUB_REPO>>` |
| Main branch | `<<MAIN_BRANCH>>` |
| Changeset parameter name | `pull_number` |
| Changeset URL pattern | `{BASE_URL}/pull/{number}` |

## Operations

### fetch-changeset

Fetch pull request metadata.

- **Tool:** `mcp_io_github_git_pull_request_read`
- **Parameters:**
  - `method`: `"get"`
  - `owner`: Repository owner (from Constants)
  - `repo`: Repository name (from Constants)
  - `pullNumber`: Integer PR number (extracted from URL)
- **Response fields:**
  - `title` → PR title
  - `body` → PR description
  - `head.ref` → source branch
  - `base.ref` → target branch
  - `state` → open / closed / merged
  - `user.login` → author
  - `labels[]` → array of labels

### fetch-changeset-diff

Fetch file-level diffs for a PR.

- **Tool:** `mcp_io_github_git_pull_request_read`
- **Parameters:**
  - `method`: `"get_files"`
  - `owner`: Repository owner
  - `repo`: Repository name
  - `pullNumber`: Integer PR number
- **Response fields:**
  - Array of files with: `filename`, `status` (added/modified/removed), `patch`, `additions`, `deletions`

### fetch-changeset-discussions

Fetch review threads on a PR.

- **Tool:** `mcp_io_github_git_pull_request_read`
- **Parameters:**
  - `method`: `"get_review_comments"`
  - `owner`: Repository owner
  - `repo`: Repository name
  - `pullNumber`: Integer PR number
- **Response fields:**
  - Array of review threads; each thread has `isResolved`, `isOutdated`, and nested comments with `body`, `user.login`, `path`, `line`

### create-pr

Open a new pull request.

- **Tool:** `mcp_io_github_git_create_pull_request`
- **Parameters:**
  - `owner`: Repository owner (from Constants)
  - `repo`: Repository name (from Constants)
  - `title`: PR title
  - `body`: PR description in Markdown
  - `head`: Source branch name
  - `base`: Target branch (use Main branch from Constants)
- **Response fields:**
  - `number` → PR number
  - `html_url` → PR URL

### post-changeset-comment

Post a comment on a PR.

- **Tool:** `mcp_io_github_git_add_issue_comment`
- **Parameters:**
  - `owner`: Repository owner
  - `repo`: Repository name
  - `issue_number`: Integer PR number (PRs are issues in GitHub)
  - `body`: Comment text in Markdown

### update-changeset

Update PR fields (e.g. description).

- **Tool:** `mcp_io_github_git_update_pull_request`
- **Parameters:**
  - `owner`: Repository owner
  - `repo`: Repository name
  - `pullNumber`: Integer PR number
  - `body`: Updated description in Markdown

## Output Format

GitHub PRs use **GitHub-flavored Markdown** — same as GitHub Issues.

### Ticket Reference in Descriptions

Link to issues using: `Closes #123` or `[TICKET-ID](ticket_manager_base_url/...)`

### Validation Rules

- [ ] No Jira or Textile markup
- [ ] Pure GitHub-flavored Markdown
- [ ] PR references ticket with Closes/Fixes keyword or link
- [ ] Files Changed covers all modified files
