# Design Tool: Figma (Figma MCP)

## MCP Loading

- **Load pattern:** `mcp_figma`
- **Tool prefix:** `mcp_figma___`

> **Deferred tools:** Call `tool_search_tool_regex` with the load pattern above **before** any tool call. Tools are NOT available until loaded. If no tools are found, Figma MCP is not configured — fall back to manual design input.

## Constants

| Constant | Value |
|----------|-------|
| Team ID | <!-- e.g. 123456789 --> |
| Default file key | <!-- e.g. abc123DEF456 --> |

## Operations

### fetch-design-file

Fetch a Figma file's metadata and top-level structure.

- **Tool:** `mcp_figma___get_file`
- **Parameters:**
  - `file_key`: Figma file key (from URL or Constants)
- **Response fields:**
  - `name` → file name
  - `document.children[]` → pages, each with `children[]` frames

### fetch-frame

Fetch details of a specific frame or component within a file.

- **Tool:** `mcp_figma___get_node`
- **Parameters:**
  - `file_key`: Figma file key
  - `node_id`: Node ID (e.g. `1:234`)
- **Response fields:**
  - `name` → frame/component name
  - `type` → FRAME / COMPONENT / INSTANCE
  - `children[]` → nested layers
  - `absoluteBoundingBox` → position and dimensions
  - `fills[]` → fill colors/gradients
  - `strokes[]` → stroke styles
  - `effects[]` → shadows, blurs

### fetch-component-styles

Fetch shared styles and components from a file.

- **Tool:** `mcp_figma___get_file_styles`
- **Parameters:**
  - `file_key`: Figma file key
- **Response fields:**
  - Array of styles with `key`, `name`, `style_type` (FILL, TEXT, EFFECT, GRID)

### fetch-images

Export frames/nodes as images.

- **Tool:** `mcp_figma___get_images`
- **Parameters:**
  - `file_key`: Figma file key
  - `ids`: Comma-separated node IDs
  - `format`: `png` | `svg` | `jpg` | `pdf`
  - `scale`: Numeric scale (e.g. `2` for 2x)
- **Response fields:**
  - `images` → map of node ID → image URL

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
