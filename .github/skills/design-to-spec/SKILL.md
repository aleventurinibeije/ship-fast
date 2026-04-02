---
name: design-to-spec
description: 'Convert design files into structured component specifications. Fetches design data via the configured design-tool provider, reads frontend best-practices, and produces a component spec document. Use when asked to "design to spec", "extract design", "component spec", or given a design URL.'
---

# Design to Spec

## When to Use

Use this skill to convert design files (Figma, Sketch, etc.) into structured component specifications that guide frontend implementation.

**Input:** Design URL, file key, or frame/component reference
**Output:** `docs/AI-analysis-plan-docs/{TICKET-ID}/DESIGN-SPEC-{TICKET-ID}.md`

---

## Prerequisites

Read before starting:
- `.github/instructions/core.instructions.md`
- `.github/context/providers/design-tool.md` — **REQUIRED:** MCP tools, operations, constants, output format
- `.github/context/architecture.md` — module layout
- `.github/context/best-practices/frontend.md` — component conventions

---

## Steps

### Step 0 — Load Provider and MCP Tools (MANDATORY FIRST STEP)

1. Read `.github/context/providers/design-tool.md`
2. If the file is a placeholder (contains only HTML comments), **no design MCP is configured** — ask the user to describe the design or provide screenshots, then proceed to Step 4 manually
3. If configured, extract the **Load pattern** from `## MCP Loading` and call:

```
tool_search_tool_regex
  pattern: {load_pattern_from_provider}
```

Wait for results. If no tools are found, the MCP server is not available — fall back to manual input.

4. Extract **Constants** for use in subsequent steps.

---

### Step 1 — Parse Design Input

From user input, extract:
- **File key** — from URL or direct input (e.g., Figma URL: `https://www.figma.com/file/{file_key}/...`)
- **Node/frame ID** — if a specific frame or component is targeted (e.g., `?node-id=1:234`)
- **Ticket ID** — if associated with a ticket (scan for ticket pattern from `context/providers/ticket-manager.md`)

If no ticket ID, use a descriptive folder name (e.g., `DESIGN-{component-name}`).

### Step 2 — Fetch Design Data

Using the design-tool provider's operations:

**2a. Fetch file structure:**
Call the **fetch-design-file** operation to get the file's top-level page/frame structure.

**2b. Fetch target frames/components:**
For each target frame or component, call the **fetch-frame** operation to get:
- Layout properties (direction, gap, padding)
- Visual properties (fills, strokes, effects)
- Dimensions and constraints
- Children/nested layers

**2c. Fetch shared styles** (optional):
Call the **fetch-component-styles** operation to get the design system's shared styles (colors, typography, effects).

### Step 3 — Read Frontend Conventions

Read `.github/context/best-practices/frontend.md` to understand:
- Component structure conventions
- Naming patterns
- State management approach
- Styling approach (CSS modules, Tailwind, styled-components, etc.)

### Step 4 — Produce Component Specification

Using the `## Output Format` from the design-tool provider config, produce a structured spec for each component:

```markdown
# Design Spec: {Design/Feature Name}

> **Ticket:** {TICKET-ID} (if applicable)
> **Design source:** [{file name}]({design_url})
> **Date:** {YYYY-MM-DD}

---

## Component Hierarchy

{Tree or list showing the component breakdown}

---

## Components

### {ComponentName}

**Figma source:** [{frame name}]({frame_url})

#### Layout
- Type: {Flex / Grid / Absolute}
- Direction: {row / column}
- Gap: {value}
- Padding: {top right bottom left}

#### Dimensions
- Width: {value or constraint}
- Height: {value or constraint}

#### Visual
- Background: {color/gradient — use design token if available}
- Border: {width style color}
- Border radius: {value}
- Shadow: {description}

#### Typography (if text element)
- Font: {family}
- Size / Weight / Line height / Color

#### States
- Default / Hover / Active / Disabled

#### Props (suggested)
| Prop | Type | Default | Description |
|------|------|---------|-------------|
| {name} | {type} | {default} | {what it controls} |

#### Children
| Element | Type | Key Properties |
|---------|------|----------------|
| {name} | {type} | {notable props} |

---

## Design Tokens Used

| Token | Value | Usage |
|-------|-------|-------|
| {name} | {value} | {where used} |

---

## Implementation Notes

{Mapping between design elements and frontend framework concepts.
 E.g., "Use `<Flex>` component for the main layout" or "This maps to a `Card` variant."}

---

## References

- Design file: [{name}]({url})
- Analysis doc: [{TICKET-ID}]({path}) (if exists)
- Frontend best practices: `.github/context/best-practices/frontend.md`
```

### Step 5 — Cross-Reference with Analysis Doc (optional)

If an analysis doc exists at `docs/AI-analysis-plan-docs/{TICKET-ID}/ANALYSIS-{TICKET-ID}.md`:
- Compare the design spec with the analysis to identify gaps
- Note any discrepancies between design and spec requirements

### Step 6 — Write the File

Write to `docs/AI-analysis-plan-docs/{TICKET-ID}/DESIGN-SPEC-{TICKET-ID}.md`.

### Step 7 — Validate

- [ ] Every component has a design source link (if MCP was available)
- [ ] Layout type and direction specified for each component
- [ ] Colors reference design tokens where available
- [ ] Interactive states documented for interactive elements
- [ ] Props table includes types, defaults, and descriptions
- [ ] Implementation Notes map design to frontend conventions
