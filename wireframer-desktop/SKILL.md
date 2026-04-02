---
name: wireframer
description: "AI wireframe generator for websites and web apps. Use this skill whenever the user wants to wireframe, sketch, lay out, or structure a website, landing page, web app, or any screen-level UI concept. Triggers on: 'wireframe', 'wireframe this', 'sketch a layout', 'lay out a page', 'structure this site', 'build a wireframe', 'lo-fi mockup', 'page structure', 'section layout', or any request to visually map out a website or app idea before high-fidelity design. Also triggers when the user describes a product/startup idea and wants to see what the site could look like, or asks for a 'quick layout' or 'rough structure' of a page. Even if they don't say 'wireframe' explicitly — if they're describing sections of a website and want to see it structured, use this skill."
---

# Wireframer

Generates structured, lo-fi wireframes from a product or page idea. Outputs to Figma (via figma-console MCP), Paper MCP, Pencil MCP, or HTML (artifact or downloadable file). After every generation, runs an interactive modification loop so the user can refine section by section until the wireframe is done.

This skill combines website structure knowledge from the `design-engineer` and `frontend-design` skills to produce wireframes that reflect real, proven page architectures — not random placeholder boxes.

---

## Core Workflow

### 1. Capture the Idea

When the user describes what they want to wireframe, extract:

- **What is it?** — Landing page, SaaS marketing site, web app dashboard, portfolio, etc.
- **Who is it for?** — Target audience / user persona
- **What's the primary goal?** — Convert signups, explain a product, showcase work, etc.
- **How many pages?** — Single page or multi-page? Which pages?

If the user gives a vague prompt like "wireframe a landing page for my AI tool", don't over-ask. Infer reasonable defaults and generate. You can always refine in the modification loop. Bias toward action.

### 2. Ask for Output Target

Before generating, always ask the user which output they want. Present these options every time:

- **Figma** — Actual Figma nodes via figma-console MCP (frames, rectangles, text layers)
- **Paper** — Via Paper MCP
- **Pencil** — Via Pencil MCP
- **HTML (in-chat)** — Rendered wireframe artifact in conversation
- **HTML (file)** — Downloadable .html file

Use the ask_user_input tool to present this as a single-select choice.

#### MCP Availability Check

After the user selects an output target, use `tool_search` to verify the required MCP tools are available. If the tools are NOT found:

1. Tell the user the MCP isn't connected yet.
2. Provide setup instructions based on the selected MCP:

**Figma (figma-console MCP):**
- Claude.ai: Not natively available. Requires the Figma Desktop Bridge plugin + figma-console MCP server. Install via Claude.ai Settings → Features → MCP Servers → Add the figma-console server URL.
- Claude Code: Add to `.claude/settings.json` under `mcpServers`. See figma-console docs for the server config.

**Paper MCP:**
- Claude.ai: Go to Settings → Features → MCP Servers → Add the Paper MCP server URL. You may need to get the URL from the Paper MCP provider's documentation.
- Claude Code: Add to `.claude/settings.json` under `mcpServers` with the Paper MCP server URL and transport config.

**Pencil MCP:**
- Claude.ai: Go to Settings → Features → MCP Servers → Add the Pencil MCP server URL. Check the Pencil MCP provider's docs for the correct URL.
- Claude Code: Add to `.claude/settings.json` under `mcpServers` with the Pencil MCP server URL and transport config.

3. After showing setup instructions, offer to switch to an available output target using `ask_user_input`. Always include HTML options as fallbacks since they require no MCP.

### 3. Generate the Section Plan

Before building anything, define the section architecture using the component registry (`references/components.json`).

The component registry is the single source of truth for how every section looks. Each component has an `id`, `name`, `category`, `variants` (light/dark), `layout`, and `children` structure. When generating a section plan, reference component IDs from the registry.

Example section plan:

```
SECTION PLAN: [Page Name]
1. nav-bar (variant: light) — Logo left, nav links right, CTA button
2. hero-split (variant: dark) — Headline, subhead, CTAs, product visual
3. logo-bar (variant: light) — "Trusted by" + 6 logo placeholders
4. features-cards (variant: light) — 3 cards with icon, title, description
5. how-it-works (variant: light) — 3-step numbered flow
6. testimonials-cards (variant: light) — 2 quote cards with avatars
7. cta-banner (variant: dark) — Centered headline + CTA
8. footer (variant: dark) — Logo, nav columns, legal links
```

If the user requests a section that does NOT exist in the registry, generate it using the design constants from `_constants` in `components.json`, then follow the "Save New Component" flow (see below).

For page-type recommendations (which components fit which page type), see `references/section-library.md`.

Briefly show the plan to the user before building. Don't wait for approval unless they seem uncertain — just show it and start building.

### 4. Generate the Wireframe

Build the wireframe in the chosen output target. Follow these rules regardless of output:

#### Design Constants (from `_constants` in components.json)

All wireframe output MUST follow these constants. No exceptions.

**Colors — strictly achromatic:**
- Light range: `#FFFFFF`, `#F5F5F5`, `#DFDFDF` (can shade darker but not too dark)
- Dark range: `#121212`, `#2C2C2C`, `#464646` (can shade lighter but not too light)
- Text: always `#FFFFFF` on dark backgrounds, `#000000` on light backgrounds
- **NO chromatic color.** No blue, green, orange, red, purple, or any hue. Only black, white, and gray.

**Fonts:**
- Headings + body text: **Geist**
- Eyebrow text, buttons, labels, code: **Geist Mono**

**Image placeholders:**
- Always use a dashed border (`1px dashed #DFDFDF`)
- Centered dimension text in Geist Mono (e.g., "1200×300", "48×48")
- Background: transparent

**Hard rules:**
- Never use shadow
- Never use effects or texture
- Never use chromatic color
- Never use lorem ipsum — use realistic placeholder text

**Additional structure rules:**
- Placeholder content, not lorem ipsum — Use realistic but generic text
- Typography hierarchy — Use size and weight to show hierarchy. H1 > H2 > H3 > Body > Caption
- 4pt grid — All spacing in 4pt increments
- 12-column grid — Desktop wireframes use 12-col, 48px margin, 24px gutter
- Annotations — Label each section clearly (e.g., "[HERO]", "[FEATURES]") so they're identifiable during the modification loop

#### Spacing Reference (4pt grid)

| Context | Desktop | Tablet | Mobile |
|---------|---------|--------|--------|
| Section padding-y | 96px | 64px | 48px |
| Section internal gap | 48px | 32px | 32px |
| Hero padding-y | 128px | 96px | 64px |
| Header height | 64px | 56px | 56px |
| Card padding | 24px | 24px | 16px |
| Card grid gap | 24px | 24px | 16px |
| Column gutter | 24px | 20px | 16px |
| Page margin | 48px | 32px | 16px |

#### Desktop canvas: 1440px wide. Mobile: 375px wide.

Generate desktop wireframe by default. If the user asks for responsive, generate desktop + mobile side by side.

---

### 5. Output-Specific Instructions

#### Figma (figma-console MCP)

Read the available figma-console tools before building. The core tools you'll use:

1. `figma_create_child` — Create frames, rectangles, text nodes inside a parent
2. `figma_set_text` — Set text content on text nodes
3. `figma_set_fills` — Set grayscale fills on shapes
4. `figma_resize_node` — Size elements correctly
5. `figma_move_node` — Position elements on the canvas
6. `figma_rename_node` — Label sections clearly (e.g., "Section: Hero", "Section: Features")
7. `figma_set_strokes` — Add borders/outlines for wireframe boxes

**Build strategy:**
- Create a top-level frame at 1440px wide, auto-height
- For each section, create a child frame with the section name
- Inside each section frame, create the wireframe elements (rectangles for images, text nodes for copy, smaller frames for cards/columns)
- Use `figma_rename_node` on every section frame so they're labeled in the layers panel
- After building, use `figma_get_selection` or `figma_take_screenshot` to confirm the output

**Naming convention:** `Wireframe / [Page Name] / [Section Name]`

#### Paper MCP

Use the Paper MCP tools to generate the wireframe. Follow the same section plan and visual rules. Adapt to whatever primitives Paper MCP provides (shapes, text, connectors). Use tool_search to discover available Paper MCP tools before generating.

#### Pencil MCP

Use the Pencil MCP tools to generate the wireframe. Follow the same section plan and visual rules. Use tool_search to discover available Pencil MCP tools before generating.

#### HTML (in-chat artifact)

Generate a single self-contained HTML file rendered as an artifact. Rules:

- Use inline CSS, no external dependencies
- Wireframe palette only (CSS variables matching `_constants` from components.json — no chromatic color)
- Fonts: Geist for headings/body, Geist Mono for eyebrow/buttons/labels (import from Google Fonts or CDN)
- Each section wrapped in a `<section>` tag with a `data-section="[name]"` attribute and a visible label
- Responsive: use CSS grid or flexbox, media queries for mobile
- Image placeholders: gray divs with centered text showing dimensions
- Add a thin 1px dashed border around each section for visual separation
- Include section labels as small caps text pinned to top-left of each section

Follow the `frontend-design` skill's approach but strip away all aesthetic flourish. This is wireframe, not UI design.

#### HTML (downloadable file)

Same as HTML in-chat, but save to `/mnt/user-data/outputs/wireframe-[page-name].html` and present via `present_files`.

---

### 6. QA Check (Automatic)

Immediately after generating the wireframe, run a QA pass before entering the modification loop. This is automatic — don't ask the user if they want QA, just do it.

**For Figma output:** Use `figma_capture_screenshot` or `figma_take_screenshot` to get a visual of the generated wireframe. Then analyze the screenshot for:

**For HTML output:** Review the generated HTML code directly.

**QA Checklist — check every item:**

1. **Alignment** — Are all elements aligned to the grid? No stray offsets? Columns line up? Text blocks left-aligned consistently within sections?
2. **Spacing** — Does section padding match the 4pt grid spacing reference? Are internal gaps consistent within and across sections? No cramped or overly loose areas?
3. **Padding & Margin** — Card padding consistent? Page margins at 48px (desktop)? Section padding-y at 96px (desktop)?
4. **Layout** — Do multi-column sections (features, pricing cards, etc.) have equal column widths? Are elements properly centered when they should be?
5. **Color compliance** — Only achromatic palette used (white/gray/black from `_constants`)? No chromatic colors anywhere? Text contrast correct (#FFFFFF on dark, #000000 on light)?
6. **Typography hierarchy** — Clear H1 > H2 > H3 > Body size progression? Consistent font sizing across same-level elements?
7. **Section labels** — Every section annotated with a visible label?
8. **Image placeholders** — All image placeholders have dashed border (1px dashed) + centered dimension text in Geist Mono? No solid-filled rectangles?
9. **Content** — No lorem ipsum? Placeholder text is realistic and appropriate length?
10. **Completeness** — All sections from the section plan are present? Nothing missing or duplicated?

**QA Output Format:**

After running the check, report results briefly to the user:

```
QA PASS ✓ — All checks passed. Wireframe is clean.
```

or

```
QA ISSUES FOUND:
• [Spacing] Features section cards have 16px gap instead of 24px — fixing now
• [Alignment] CTA button is 4px off-center — fixing now
```

If issues are found, fix them immediately before entering the modification loop. Don't ask the user for permission to fix QA issues — just fix them and report what was corrected.

---

### 7. Post-Generation: Modification Loop

This is critical. After every successful generation:

1. **List all sections** — Present the sections as a numbered, selectable list using `ask_user_input` (multi-select so user can pick one or more sections to modify)
2. **Ask what to change** — For each selected section, ask the user what they want to modify. Examples: "swap to 2-column layout", "add a second CTA", "remove this section", "add a pricing table after features", "make the hero split-screen"
3. **Regenerate only that section** — Don't rebuild the entire wireframe. Modify only the targeted section in the chosen output.
4. **Re-list sections** — After modification, show the updated section list again and ask if they want to modify more or if the wireframe is done.
5. **Exit condition** — When the user says they're done, satisfied, or moves on, confirm completion and offer next steps:
   - "Want me to export this to a different format?"
   - "Ready to move to high-fidelity design?"
   - "Want me to generate the mobile version?"

**Always include an "I'm done / Looks good" option in the section selection so the user can exit the loop cleanly.**

---

## Section Library

For page-type composition recommendations (which components go in which order for which page type), see `references/section-library.md`. This maps page types to component IDs from the registry.

For the actual component definitions (layout, spacing, colors, children), see `references/components.json`. This is the single source of truth for how every section renders.

Always consult both when generating a section plan.

---

## Component Registry

The component registry (`references/components.json`) is a living file. It defines every section type the wireframer can generate.

**Structure of a component entry:**

```json
{
  "component-id": {
    "name": "Human-readable Name",
    "category": "navigation | hero | features | social-proof | process | pricing | faq | cta | footer | developer",
    "variants": {
      "light": { "background": "...", "textColor": "...", ... },
      "dark": { "background": "...", "textColor": "...", ... }
    },
    "layout": { "direction": "...", "gap": ..., "paddingX": ..., "paddingY": ... },
    "children": [ ... ]
  }
}
```

**When rendering:** Look up the component ID, apply the selected variant's colors, use the layout spec for structure, and render children by role. Always use fonts and colors from `_constants`. Always use button styles from `_buttons`.

---

## Save New Component Flow

When the user requests a section that does NOT exist in the registry:

1. **Generate it** — Use the design constants from `_constants` to create a section that follows all wireframe rules. Match the structure depth of existing components (top-level roles + types, not full recursive trees).

2. **Show it to the user** — Render the section in the chosen output target as part of the wireframe.

3. **Ask to save** — After the user approves the section (either explicitly or by not requesting changes to it during the modification loop), ask:
   - "This section isn't in the component library yet. Want me to save it as a reusable component?"

4. **If yes — save it:**
   - Generate a component ID (lowercase, kebab-case, descriptive — e.g., `comparison-table`, `team-grid`, `announcement-bar`)
   - Structure it as a proper component entry with variants (at minimum light + dark), layout, and children
   - Append it to `references/components.json`
   - Confirm: "Saved `comparison-table` to the component registry. It'll be available for all future wireframes."

5. **If no** — move on. Don't ask again.

**The registry grows over time.** Every approved custom section becomes a reusable component for future wireframes. This is how the skill learns and stays consistent.

---

## What This Skill Does NOT Do

- **No high-fidelity design** — No colors, no brand, no real images. That's the design-engineer skill's job after wireframing.
- **No copy generation** — Uses placeholder copy only. If the user wants real copy, that's a separate task.
- **No code generation beyond HTML wireframes** — This doesn't output React components, Framer pages, or production code. It outputs structure.
- **No user flow diagrams** — This is screen-level wireframing, not flow mapping. For flows, use a different tool.

---

## Response Protocol

1. Capture the idea (brief, don't over-interview)
2. Ask for output target (Figma / Paper / Pencil / HTML in-chat / HTML file)
3. Verify MCP availability — if missing, show setup instructions + offer fallback
4. Show the section plan
5. Generate the wireframe
6. Run QA check — fix any issues automatically
7. Enter modification loop
8. Exit when user is satisfied
