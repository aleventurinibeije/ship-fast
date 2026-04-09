# Provider Loading Protocol

> **Reference protocol for all skills that use external providers (ticket-manager, code-review, design-tool).**  
> Skills include this block in their Step 0 instead of duplicating it.

---

## Standard Provider Loading Steps

For each provider a skill needs:

1. Read the provider config file (e.g. `.github/context/providers/ticket-manager.md`)
2. If the file is a placeholder (contains only HTML comments or a "N/A" banner) — **no provider is configured**. Skip all MCP steps for that provider; collect input manually or from local files instead.
3. Extract the **Load pattern** from the `## MCP Loading` section of the provider config.
4. Call `tool_search_tool_regex` with that pattern:
   ```
   tool_search_tool_regex
     pattern: {load_pattern_from_provider}
   ```
   **Wait for results before proceeding.** Only after this call completes can you call any tools returned by the search.
5. Extract **Constants** (project ID, base URL, ticket pattern, status IDs, tracker IDs, etc.) from the provider config's `## Constants` section. Store them for use in all subsequent steps.

> If two providers are needed (e.g. both ticket-manager and code-review), both `tool_search_tool_regex` calls can be made **in parallel** — but only after both provider config files have been read.

---

## Provider Config Files

| Provider | Config file |
|---|---|
| Ticket manager | `.github/context/providers/ticket-manager.md` |
| Code review | `.github/context/providers/code-review.md` |
| Design tool | `.github/context/providers/design-tool.md` — optional |

Each provider config file documents the MCP load pattern, constants, and operation specs for that integration.  
See `stack/providers/` for the available provider templates to choose from during bootstrap.
