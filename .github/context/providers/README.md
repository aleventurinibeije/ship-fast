# context/providers/

<!-- This folder holds the active provider configurations for external integrations.
     Files here are referenced by ALL agents, skills, and instructions
     using the fixed filenames below.

     PROVIDER SLOT MODEL:
     Each external integration is a "slot" filled by exactly one provider config.
     Skills read these configs at runtime to discover MCP tool names, parameters,
     and output format rules — they NEVER hardcode MCP tool references.

     SETUP STEPS:
     1. Go to the .github/stack/providers/ folder
     2. Copy the variant that matches your tools:
           cp stack/providers/ticket-jira.md     context/providers/ticket-manager.md
           cp stack/providers/review-github.md   context/providers/code-review.md
           cp stack/providers/design-figma.md    context/providers/design-tool.md
     3. Edit the copied files — fill in your project-specific constants.
     Or run `@01-bootstrap` — it collects constants from you and writes these files automatically.

     ACTIVE FILES (must use these exact names):
     - ticket-manager.md  ← ticket/issue tracker (Jira, Redmine, GitHub Issues, etc.)
     - code-review.md     ← merge/pull request platform (GitLab, GitHub, Bitbucket, etc.)
     - design-tool.md     ← design tool (Figma, Sketch, etc.) — OPTIONAL

     If your project doesn't use a design tool, leave design-tool.md as the placeholder.
     If you have no MCP integration for a slot, leave the placeholder — skills will
     gracefully degrade and ask for manual input instead.

     HOW SKILLS CONSUME PROVIDERS:
     1. Read the provider config file (e.g. context/providers/ticket-manager.md)
     2. Extract the MCP load pattern from the "## MCP Loading" section
     3. Call tool_search_tool_regex with that pattern to load deferred tools
     4. Call the operation's tool name with the documented parameters

     PROVIDER CONFIG FORMAT (each file follows this structure):
     ──────────────────────────────────────────────────
     # {Slot}: {ProviderName}

     ## MCP Loading
     - Load pattern: `{regex_pattern_for_tool_search_tool_regex}`
     - Tool prefix: `{tool_name_prefix}`

     ## Constants
     | Constant | Value |
     |----------|-------|
     | {NAME}   | {value or <<PLACEHOLDER>>} |

     ## Operations

     ### {operation-name}
     - **Tool:** `{full_mcp_tool_name}`
     - **Parameters:**
       - `param_name`: {description or mapping}
     - **Response fields:**
       - `field.path` → {what it contains}

     ## Output Format
     {Format rules specific to this provider — used by description-writing skills}
     ──────────────────────────────────────────────────
-->
