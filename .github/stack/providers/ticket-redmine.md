# Ticket Manager: Redmine

> **Integration: REST API** — Uses the [Redmine REST API](https://www.redmine.org/projects/redmine/wiki/Rest_api) directly via `run_in_terminal` (curl). No MCP server required.

## Constants

| Constant | Value |
|----------|-------|
| Project identifier | `<<REDMINE_PROJECT_ID>>` |
| Base URL | `<<REDMINE_BASE_URL>>` <!-- e.g. https://redmine.example.com --> |
| Ticket pattern | `#\d+` |
| API key env var | `<<REDMINE_API_KEY>>` |
| Tracker ID: parent ticket | `9` <!-- Epica --> |
| Tracker ID: child ticket | `13` <!-- Task --> |
| Status ID: Open | `12` |
| Status ID: In Analysis | `9` |
| Status ID: In Development | `26` |
| Status ID: In Review | `17` |

## Authentication

All requests must include the API key as an HTTP header:

```
X-Redmine-API-Key: $REDMINE_API_KEY
```

The API key is found at `/my/account` when logged in. Store it in the `REDMINE_API_KEY` environment variable — never hardcode it.

> Enable the REST API in Redmine: **Administration → Settings → API → Enable REST API**.

## Operations

### fetch-ticket

Fetch an issue's full details.

- **Method:** `GET`
- **Endpoint:** `<<REDMINE_BASE_URL>>/issues/[id].json?include=journals`
- **curl example:**
  ```sh
  curl -s -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
    "<<REDMINE_BASE_URL>>/issues/[id].json?include=journals"
  ```
- **Response fields:**
  - `issue.subject` → ticket title
  - `issue.description` → full description
  - `issue.tracker.name` → Bug / Feature / Task
  - `issue.status.name` → current status
  - `issue.assigned_to.name` → assignee
  - `issue.journals` → comment history (when `include=journals`)

### create-ticket

Create a new issue.

- **Method:** `POST`
- **Endpoint:** `<<REDMINE_BASE_URL>>/issues.json`
- **Headers:** `Content-Type: application/json`
- **curl example:**
  ```sh
  curl -s -X POST \
    -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"issue": {"project_id": "<<REDMINE_PROJECT_ID>>", "subject": "...", "description": "...", "tracker_id": <<TRACKER_ID_CHILD>>, "status_id": <<STATUS_ID_OPEN>>}}' \
    "<<REDMINE_BASE_URL>>/issues.json"
  ```
- **Body fields (`issue` object):**
  - `project_id`: Project identifier (string)
  - `subject`: Issue title
  - `description`: Issue body (Textile markup)
  - `tracker_id`: `1` = Bug, `2` = Feature, `3` = Task
  - `assigned_to_id`: User ID to assign (optional)
  - `priority_id`: Priority ID (optional)
- **Response fields:**
  - `issue.id` → created issue ID

### update-ticket

Update an issue's fields or add a comment.

- **Method:** `PUT`
- **Endpoint:** `<<REDMINE_BASE_URL>>/issues/[id].json`
- **Headers:** `Content-Type: application/json`
- **curl example:**
  ```sh
  curl -s -X PUT \
    -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"issue": {"notes": "Comment text", "status_id": 2}}' \
    "<<REDMINE_BASE_URL>>/issues/[id].json"
  ```
- **Body fields (`issue` object):**
  - `notes`: Comment text (optional)
  - `private_notes`: `true` to make notes private (optional)
  - `status_id`: New status ID (optional)
  - `assigned_to_id`: Reassign to user ID (optional)
  - `subject`, `description`, `priority_id`, `tracker_id`: Other updatable fields

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
