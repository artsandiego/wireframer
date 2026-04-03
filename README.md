# Wireframer

AI wireframe generator skill for Claude. It outputs structured, lo-fi wireframes to Figma, Paper MCP, Pencil MCP, or HTML, driven by a component registry that keeps every wireframe consistent.

Built for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [Claude Desktop](https://claude.ai).

# What it does

- Generates wireframes from a product or page idea
- Uses a component registry (`components.json`) so output is consistent every time
- Supports multiple output targets: Figma, Paper MCP, Pencil MCP, and HTML
- Runs a QA subagent that checks after every generation (alignment, spacing, color, typography)
- Interactive modification loop — refine section by section until done
- Saves new custom sections to the registry after approval, so the library grows over time

# Installation

## Option 1: Claude Code CLI (Recommended)

### Install via curl:

1. Open your terminal from your project folder.
> 🚨 Make sure your terminal is open in the project where you want to install the `/wireframer` command.

2. Run this one-line installer:

```bash
curl -fsSL https://raw.githubusercontent.com/artsandiego/wireframer/main/install.sh | bash
```

This creates the following in your project:

```
your-project/
├── .claude/
│   ├── commands/
│   │   └── wireframer.md              ← the command
│   └── wireframer-refs/
│       ├── components.json             ← component registry
│       └── section-library.md          ← page-type compositions
```

3. Open Claude Code in that project and run:

```
/wireframer
```

### Manual Install:

If you prefer not to use the install script, download the files manually:

1. Create these directories inside your project folder:
```
your-project/
├── .claude/
│   ├── commands/ 
│   └── wireframer-refs/
```

2. Clone this repo or download it as a ZIP file.

3. Move these files from the cloned repo or downloaded ZIP into your project folder:
- `wireframer.md` to `.claude/commands`
- `components.json` and `section-library.md` to `.claude/wireframer-refs`

Your directory structure should look like this:
```
your-project/
├── .claude/
│   ├── commands/
│   │   └── wireframer.md            
│   └── wireframer-refs/
│       ├── components.json            
│       └── section-library.md      
```

Then open Claude Code and type:

```
/wireframer
```

</details>

## Option 2: Claude Desktop App

The Claude Desktop app uses `.skill` files, which you install through the UI.

1. Clone this repo or download as ZIP and get the `wireframer.skill` file.

2. The `.skill` file is just a zip archive containing:

```
wireframer.skill
└── wireframer/
    ├── SKILL.md                        ← the skill definition
    └── references/
        ├── components.json             ← component registry (same file)
        └── section-library.md          ← page-type compositions (same file)
```

3. To install, open Claude Desktop → Customize → Skills → Create Skills → Upload a skill, then drag and drop the `wireframer.skill` file or click "Add Skill" and select it.

### Building the `.skill` file yourself

If you want to modify the skill and rebuild the `.skill` file:

1. Make your changes to the files in `claude-desktop/wireframer/`
2. Zip the `wireframer/` folder:

```bash
cd claude-desktop
zip -r wireframer.skill wireframer/
```

3. Install the `.skill` file in Claude Desktop as described above.


# Repo structure

```
wireframer/
├── README.md
├── install.sh                          ← curl installer for Claude Code
├── claude-code/                        ← Claude Code version
│   ├── commands/
│   │   └── wireframer.md
│   └── wireframer-refs/
│       ├── components.json
│       └── section-library.md
├── claude-desktop/                     ← Claude Desktop version
│   └── wireframer/
│       ├── SKILL.md
│       └── references/
│           ├── components.json
│           └── section-library.md
└── wireframer.skill                    ← pre-built .skill file (for releases)
```

The `components.json` and `section-library.md` files are identical across both versions (`claude-desktop` and `claude-code`). If you add a component in one, copy it to the other to keep them in sync.


# Design constants

All wireframes follow strict rules:

- **Colors**: Achromatic only — `#FFFFFF`, `#F5F5F5`, `#DFDFDF`, `#121212`, `#2C2C2C`, `#464646`. No chromatic colors.
- **Fonts**: Geist (headings + body), Geist Mono (eyebrow, buttons, labels)
- **Images**: Dashed border, dimension label centered, fill contrasts with background
- **Rules**: No shadows, no effects, no texture. 4pt grid. 12-column layout.

# Component registry

The skill is powered by `components.json` — a structured library of section definitions. Each component has:

- **Variants** (light/dark) with full color specs
- **Layout** definitions (direction, gap, padding)
- **Children** with roles (logo, headline, cta, card-grid, etc.)
- **Font assignments** (Geist or Geist Mono per role)

## Built-in components

| ID | Name | Category |
|----|------|----------|
| `nav-bar` | Navigation Bar | navigation |
| `hero-split` | Hero — Split Layout | hero |
| `hero-centered` | Hero — Centered | hero |
| `logo-bar` | Logo Bar | social-proof |
| `features-cards` | Features — Card Grid | features |
| `features-alternating` | Features — Alternating Rows | features |
| `how-it-works` | How It Works — Steps | process |
| `testimonials-cards` | Testimonials — Card Grid | social-proof |
| `pricing-cards` | Pricing — Tier Cards | pricing |
| `faq-accordion` | FAQ — Accordion | faq |
| `cta-banner` | CTA Banner | cta |
| `metrics-bar` | Metrics Bar | social-proof |
| `code-block-section` | Code Block Section | developer |
| `about-split` | About — Split (Image + Text) | about |
| `footer` | Footer | footer |

## Adding custom components

When you request a section that doesn't exist in the registry, Wireframer generates it on the fly. After you approve the output, it asks if you want to save it as a reusable component. If you say yes, it appends the section to `components.json` with light and dark variants, making it available for all future wireframes.

# Usage

1. Describe what you want to wireframe (a landing page, portfolio, etc.)
2. Pick an output target (Figma, Paper, Pencil, HTML)
3. The wireframer shows a section plan using component IDs from the registry
4. It generates the wireframe
5. The QA check runs automatically
6. You pick sections to modify, or confirm you're done
7. Any new custom sections can be saved to the registry

# License

MIT
