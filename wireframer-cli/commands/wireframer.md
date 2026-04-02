# /wireframer

AI wireframe generator for websites and web apps. Generates structured, lo-fi wireframes from a product or page idea with consistent output driven by a component registry.

## Setup

This command requires two reference files in your project:

- `.claude/wireframer-refs/components.json` — The component registry (design constants, section definitions, button specs)
- `.claude/wireframer-refs/section-library.md` — Page-type composition guide (which components go where)

Both files are the single source of truth for wireframe output. Always read them before generating.

## Core Workflow

### 1. Capture the Idea

Extract from the user's prompt:

- **What is it?** — Landing page, SaaS site, dashboard, portfolio, etc.
- **Who is it for?** — Target audience
- **Primary goal?** — Convert signups, explain product, showcase work
- **How many pages?** — Single or multi-page

Don't over-ask. Infer reasonable defaults and generate. Refine in the modification loop.

### 2. Ask for Output Target

Ask the user which output they want:

- **Figma** — Via figma-console MCP (actual Figma nodes: frames, rectangles, text layers)
- **Paper** — Via Paper MCP
- **Pencil** — Via Pencil MCP
- **HTML file** — Save as `.html` file in the project

After the user selects, verify the MCP tools are available. If not, inform them:

**Figma:** Requires figma-console MCP. Add to `.claude/settings.json` under `mcpServers`.
**Paper:** Requires Paper MCP. Add to `.claude/settings.json` under `mcpServers`.
**Pencil:** Requires Pencil MCP. Add to `.claude/settings.json` under `mcpServers`.
**HTML:** Always available, no MCP needed.

If the selected MCP isn't available, offer to switch. HTML is always the fallback.

### 3. Generate the Section Plan

Read `.claude/wireframer-refs/components.json` and `.claude/wireframer-refs/section-library.md`.

Reference component IDs from the registry to build the section plan:

```
SECTION PLAN: [Page Name]
1. nav-bar (variant: light) — Logo, nav links, CTA
2. hero-split (variant: dark) — Headline, subhead, CTAs, product visual
3. logo-bar (variant: light) — "Trusted by" + logo placeholders
4. features-cards (variant: light) — 3 cards with icon, title, description
5. cta-banner (variant: dark) — Centered headline + CTA
6. footer (variant: dark) — Logo, nav columns, legal links
```

If the user requests a section NOT in the registry, generate it using the `_constants` from `components.json`, then follow the Save New Component flow.

Show the plan briefly before building. Don't wait for approval unless uncertain.

### 4. Design Constants (MANDATORY)

Read from `_constants` in `components.json`. These are non-negotiable:

**Colors — strictly achromatic:**
- Light range: `#FFFFFF`, `#F5F5F5`, `#DFDFDF` (can shade darker but not too dark)
- Dark range: `#121212`, `#2C2C2C`, `#464646` (can shade lighter but not too light)
- Text: always `#FFFFFF` on dark, `#000000` on light
- **NO chromatic color.** No blue, green, orange, red, purple. Only black, white, gray.

**Fonts:**
- Headings + body: **Geist**
- Eyebrow, buttons, labels, code: **Geist Mono**

**Image placeholders:**
- Must have visible fill contrasting with section background
- On light sections: background `#DFDFDF`, border `1px dashed #DFDFDF`, label color `#464646`
- On dark sections: background `#464646`, border `1px dashed #464646`, label color `#DFDFDF`
- Centered dimension text in Geist Mono (e.g., "1200×300")
- Never transparent

**Hard rules:**
- Never use shadow
- Never use effects or texture
- Never use chromatic color
- Never use lorem ipsum — realistic placeholder text only
- 4pt grid for all spacing
- 12-column grid: 1440px desktop, 48px margin, 24px gutter

### 5. Generate the Wireframe

Build in the chosen output target using the component registry specs.

**For Figma:** Use `figma_create_child`, `figma_set_text`, `figma_set_fills`, `figma_resize_node`, `figma_move_node`, `figma_rename_node`, `figma_set_strokes`. Create a 1440px-wide top-level frame, then child frames per section.

**For Paper:** Use Paper MCP tools (`write_html`, `create_artboard`). Create a 1440px artboard, then build sections via `write_html` with inline styles matching the component specs.

**For Pencil:** Use Pencil MCP tools. Adapt to available primitives.

**For HTML:** Generate a self-contained `.html` file. Use Geist/Geist Mono via CDN import. CSS variables for the color palette. Each section in a `<section>` tag with `data-section` attribute and visible label. Responsive via media queries.

### 6. QA Check (Automatic)

Run immediately after generation. Don't ask — just do it.

**QA Checklist:**

1. **Alignment** — Everything on grid? Columns line up?
2. **Spacing** — Section padding matches 4pt grid? Consistent gaps?
3. **Padding & Margin** — Card padding consistent? Page margins at 48px?
4. **Layout** — Multi-column sections have equal widths? Centered elements actually centered?
5. **Color compliance** — Only achromatic palette from `_constants`? No chromatic colors? Text contrast correct?
6. **Typography** — Clear H1 > H2 > H3 > Body hierarchy? Geist for headings/body, Geist Mono for eyebrow/buttons?
7. **Section labels** — Every section annotated with visible label?
8. **Image placeholders** — Visible fill contrasting with background? Dashed border? Dimension text in Geist Mono?
9. **Content** — No lorem ipsum? Realistic placeholder text?
10. **Completeness** — All sections from plan present?

Report briefly:
```
QA PASS ✓ — All checks passed.
```
or fix issues immediately and report what was corrected.

### 7. Modification Loop

After QA:

1. **List all sections** — Present as a selectable list. Include "I'm done" option.
2. **User picks section(s)** — Ask what to change.
3. **Regenerate only that section** — Don't rebuild everything.
4. **Re-list sections** — Repeat until user says done.
5. **Exit** — Offer next steps: export to different format, mobile version, or move to high-fidelity.

### 8. Save New Component Flow

When a section doesn't exist in the registry:

1. Generate it using `_constants`
2. Show it to the user
3. After approval, ask: "Save as a reusable component?"
4. If yes: generate a component entry with light + dark variants, append to `.claude/wireframer-refs/components.json`
5. Confirm: "Saved `[component-id]` to the registry."

The registry grows over time. Every approved custom section becomes reusable.

## What This Command Does NOT Do

- No high-fidelity design (no brand colors, no real images)
- No copy generation (placeholder text only)
- No production code (no React components, no Framer pages)
- No user flow diagrams (screen-level wireframing only)
