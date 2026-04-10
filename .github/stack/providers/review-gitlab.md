# Code Review: GitLab MR (GitLab MCP)

## MCP Loading

- **Load pattern:** `mcp_io_modelconte_gitlab`
- **Tool prefix:** `mcp_io_modelconte_gitlab-mcp___`

> **Deferred tools:** Call `tool_search_tool_regex` with the load pattern above **before** any tool call. Tools are NOT available until loaded.

## Constants

| Constant | Value |
|----------|-------|
| Project ID (numeric) | `<<GITLAB_PROJECT_ID>>` |
| Base URL | <!-- e.g. https://gitlab.example.com --> |
| Main branch | `<<MAIN_BRANCH>>` |
| Changeset parameter name | `merge_request_iid` |
| Changeset URL pattern | `{BASE_URL}/{group}/{project}/-/merge_requests/{iid}` |

> Always use the **numeric** project ID — never the project path string.

## Operations

### fetch-changeset

Fetch changeset (MR) metadata.

- **Tool:** `mcp_io_modelconte_gitlab-mcp___get_merge_request_details`
- **Parameters:**
  - `project_id`: Numeric project ID (from Constants)
  - `merge_request_iid`: Integer MR ID (extracted from URL)
- **Response fields:**
  - `title` → changeset title
  - `description` → changeset description
  - `source_branch` → source branch name
  - `target_branch` → target branch name
  - `state` → open / merged / closed
  - `author.name` → author display name
  - `labels[]` → array of labels

### fetch-changeset-diff

Fetch file-level diffs for a changeset.

- **Tool:** `mcp_io_modelconte_gitlab-mcp___get_merge_request_diff`
- **Parameters:**
  - `project_id`: Numeric project ID
  - `merge_request_iid`: Integer MR ID
- **Response fields:**
  - List of changed files with: `old_path`, `new_path`, `diff`, `new_file`, `renamed_file`, `deleted_file`

### fetch-changeset-discussions

Fetch existing review discussions on a changeset.

- **Tool:** `mcp_io_modelconte_gitlab-mcp___get_merge_request_discussions`
- **Parameters:**
  - `project_id`: Numeric project ID
  - `merge_request_iid`: Integer MR ID
- **Response fields:**
  - Array of discussion threads, each with `notes[]` containing `body`, `author.name`, `resolved`

### create-pr

Open a new merge request.

- **Tool:** `mcp_io_modelconte_gitlab-mcp___create_merge_request`
- **Parameters:**
  - `project_id`: Numeric project ID (from Constants)
  - `title`: MR title
  - `description`: MR description in Markdown
  - `source_branch`: Source branch name
  - `target_branch`: Target branch (use Main branch from Constants)
- **Response fields:**
  - `iid` → MR iid
  - `web_url` → MR URL

### post-changeset-comment

Post a comment on a changeset.

- **Tool:** `mcp_io_modelconte_gitlab-mcp___add_merge_request_comment`
- **Parameters:**
  - `project_id`: Numeric project ID
  - `merge_request_iid`: Integer MR ID
  - `body`: Comment text in Markdown

### update-changeset

Update changeset fields (e.g. description).

- **Tool:** `mcp_io_modelconte_gitlab-mcp___edit_merge_request`
- **Parameters:**
  - `project_id`: Numeric project ID
  - `merge_request_iid`: Integer MR ID
  - `description`: Updated description in Markdown

## Output Format

GitLab uses **Markdown** (GitHub-flavored with GitLab extensions).

### Key GitLab Markdown Rules

- Standard Markdown headings: `## Heading`
- Code blocks: triple backticks with language tag
- Tables: standard Markdown pipe tables
- Task lists: `- [ ] item` / `- [x] item`
- Inline code: backticks `` `code` ``
- Links: `[text](url)`

### Ticket Reference in Descriptions

Link to tickets using: `[{TICKET-ID}]({TICKET_MANAGER_BASE_URL}/browse/{TICKET-ID})`

Read the ticket manager's base URL from `context/providers/ticket-manager.md` → Constants → Base URL.

### Validation Rules

Before finishing any GitLab-format output, verify:

- [ ] No Jira markup (no `h2.`, no `{{…}}`, no `{code}`)
- [ ] Summary references ticket with a link
- [ ] Files Changed covers all modified files
- [ ] Checklist includes project-specific items
- [ ] Pure Markdown only — no wiki markup
