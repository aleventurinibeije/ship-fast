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

Follow the **Standard Provider Loading Steps** from `.github/context/providers/LOADING-PROTOCOL.md` for the **design-tool** provider (`.github/context/providers/design-tool.md`).

> This project marks design-tool as N/A (backend-only). If the provider file starts with a "Not applicable" banner, skip all MCP steps and proceed to Step 4 using manual input or screenshots.

Extract **Constants** for use in subsequent steps if provider is active.

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
For each target frame or component, call the **fetch-design-context** operation to get:
- Reference code (React+Tailwind — adapt to project stack)
- Screenshot of the rendered node
- Code Connect snippets or design annotations if present

**2c. Fetch design tokens** (optional):
Call the **fetch-variable-defs** operation to get the design system's token values (colors, spacing, typography). Map them to the project's CSS variables or theme tokens.

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
