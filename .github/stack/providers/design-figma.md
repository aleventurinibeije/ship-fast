# Design Tool: Figma (Figma MCP)

## MCP Loading

- **Load pattern:** `mcp_com_figma_mcp`
- **Tool prefix:** `mcp_com_figma_mcp_`

> **Deferred tools:** Call `tool_search_tool_regex` with the load pattern above **before** any tool call. Tools are NOT available until loaded. If no tools are found, Figma MCP is not configured — fall back to manual design input.

## Constants

| Constant | Value |
|----------|-------|
| Team ID | {{TEAM_ID}} |
| Default file key | {{DEFAULT_FILE_KEY}} |

## Operations

### fetch-design-file

Fetch a Figma file's metadata and top-level node structure (for browsing pages/frames).

- **Tool:** `mcp_com_figma_mcp_get_metadata`
- **Parameters:**
  - `fileKey`: Figma file key (from URL or Constants)
  - `nodeId` *(optional)*: Scope to a specific node; omit for the full file
- **Response:** XML document listing node IDs, types, names, positions, and sizes — use this to discover node IDs before calling `fetch-design-context`

---

### fetch-design-context

**Primary tool.** Fetch design context, reference code (React+Tailwind), and a screenshot for a specific node.

- **Tool:** `mcp_com_figma_mcp_get_design_context`
- **Parameters:**
  - `fileKey`: Figma file key
  - `nodeId`: Node ID (e.g. `1:234`) — use `"0:1"` for the root if no specific node
  - `clientFrameworks` *(optional)*: e.g. `["react"]`
  - `clientLanguages` *(optional)*: e.g. `["typescript"]`
- **Response:**
  - `code` → reference implementation in React+Tailwind (adapt to project stack)
  - `screenshot` → rendered image of the node
  - Code Connect snippets if mapped, or design annotations if present

> **Always prefer this tool** over lower-level operations for component implementation.

---

### fetch-screenshot

Get a screenshot of a specific node (without code context).

- **Tool:** `mcp_com_figma_mcp_get_screenshot`
- **Parameters:**
  - `fileKey`: Figma file key
  - `nodeId`: Node ID
- **Response:** PNG screenshot of the node

---

### fetch-variable-defs

Fetch design token / variable definitions (colors, spacing, typography, etc.).

- **Tool:** `mcp_com_figma_mcp_get_variable_defs`
- **Parameters:**
  - `fileKey`: Figma file key
- **Response:** Variable collections with resolved values — use to map design tokens to project CSS vars or theme values

## Output Format

Design specs are written in **Markdown** with structured component breakdowns.

### Component Spec Structure

```markdown
## {Component Name}

**Figma source:** [{frame name}]({figma_url})

### Layout
- Type: {Flex / Grid / Absolute}
- Direction: {row / column}
- Gap: {value}
- Padding: {top right bottom left}

### Dimensions
- Width: {value or constraint}
- Height: {value or constraint}
- Min/Max: {if applicable}

### Visual
- Background: {color/gradient}
- Border: {width style color}
- Border radius: {value}
- Shadow: {x y blur spread color}

### Typography (if text element)
- Font: {family}
- Size: {value}
- Weight: {value}
- Line height: {value}
- Color: {value}

### States
- Default: {description}
- Hover: {description}
- Active: {description}
- Disabled: {description}

### Children
| Element | Type | Key Properties |
|---------|------|----------------|
| {name}  | {type} | {notable props} |
```

### Validation Rules

Before finishing any design spec output, verify:

- [ ] Every component has a Figma source link
- [ ] Layout type and direction are specified
- [ ] Colors use design token names where available (fall back to hex)
- [ ] Typography values match the design system
- [ ] Interactive states documented where applicable
- [ ] Children table lists all direct child elements
