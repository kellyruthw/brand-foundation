# Brand Foundation

A [Claude Code](https://claude.com/claude-code) skill that turns a brand's design system into a coded foundation (tokens, mixins and surfaces) and then keeps the build from drifting away from it.

I built it to encode how I work as a design engineer, so an AI coding agent follows the same discipline I do instead of improvising.

## The problem it solves

An AI building a site without a foundation will happily invent a fourth shade of grey, hardcode a font size, or build a second card component that's 3px different from the first. Each one is small. Together they're why AI-built sites look generic and drift out of spec.

This skill makes the agent do what a careful design engineer does:

1. **Audit the design first**: find the patterns, the near-duplicates (`#22336C` vs `#22346C`) and the gaps.
2. **Confirm with the designer**: never resolve an inconsistency by silently picking one value.
3. **Encode the truth once**: variables, mixins and surface classes, then a one-page visual review before any component exists.
4. **Build component by component, reuse first**: before creating anything, check whether it already exists or can be extended. Silent one-offs are the failure mode.
5. **Sweep for drift**: a consistency pass that finds hardcoded values, sibling components, dead code and contrast failures, and reports each one by file and line, with a visual HTML page where each issue is rendered: near-duplicate colors as swatches side by side, off-grid spacing as bars, contrast failures as the real text on its real background.

## What's in it

```
brand-foundation/
├── SKILL.md                      The process: five phases, the rules, and how to report
└── references/
    ├── audit-checklist.md        What to look for in a design audit, plus the report format
    ├── variables.scss            Token template: every key a foundation needs, placeholder values
    ├── mixins.scss               Fluid type, type styles, spacing, breakpoints, motion, grid
    ├── globals.scss              Base styles and one surface class per background token
    ├── main.scss                 Entry-file order
    ├── review-artifact.md        Layout for the visual review page of the foundation
    └── findings-report.md        Layout for the visual page of consistency-pass findings
```

The templates hold **keys, not values**. Every project gets the same structure, but its colors, fonts, sizes and spacing come from its own design. Placeholders are deliberately loud (colors are magenta `#ff00ff`, fonts are named "TODO …"), so a value nobody filled in is obvious on the review page. Lookups fail the build on a typo instead of silently outputting nothing.

A few principles are built in:

- **Two tiers of color.** A raw palette named for what it is, and semantic aliases named for what they do. Components only use the semantic names, so a rebrand is a one-line change.
- **Status colors named by meaning.** Success, warning, error and info, each with a border, a fill and a text color, checked for contrast. Never "positive/negative": that naming is how a success message ends up orange.
- **Scales, not one-offs.** Type, spacing (on a base grid the designer confirms), shadows (sm / md / lg) and radii (scaled to element size) are all scales.
- **Accessibility by default.** Text contrast is checked whenever a color is chosen, and every transition respects reduced-motion settings.
- **Verify, don't eyeball.** Compile the CSS before and after a change and compare the two. That proves a refactor changed nothing, or changed exactly what was intended.

## Case study: a consistency pass on a production app

I ran the consistency pass on **heartcore-books**, a bookkeeping app I'm building in React with SCSS modules. It had about 50 style modules and an existing foundation. The foundation was in good shape: every color already went through a variable. The pass still found real drift:

| Found | What it was | Resolution |
|---|---|---|
| 26 font sizes skipping the type scale | Components typing `rem(14)` instead of the size token; one heading at an off-scale 30px | 22 routed through tokens, 4 redundant lines deleted, heading moved to the 32px step |
| A radius bug | `$radius-control - 2px`, written when the token was 6px; the token later became 4px, so a table's corners were silently 2px | Fixed to the token |
| Success messages rendering orange | A token named "positive" pointed at orange, so success, info and warning all looked alike, and warnings looked identical to errors | Status colors rebuilt by meaning: green, orange, red/pink, blue |
| Two names for one color, plus a near-duplicate | `#f5f5fc` held twice; `#dfe5f9` vs `#e5e5f9` | Aliased; designer picked one |
| 4 one-off shadows, some pure black | Each floating element had invented its own | An sm / md / lg scale, tinted with the brand navy |
| Spacing off the 4px grid | 6, 7, 9, 10 and 17px values | Snapped to the grid |
| 6 transitions ignoring reduced motion | Animations ran for users who had turned motion off | Wrapped in a motion-safe mixin |
| Dead code | 2 components nothing imported, 12 variables nothing referenced | Removed |

Findings came back in three groups: **safe fixes** (no visual change), **visual changes** (before and after listed) and **decisions for the designer** (one line each, with a recommendation). That split let the designer approve each group quickly.

Every round was checked by compiling all 52 modules before and after and comparing the output. The safe fixes produced identical CSS, and the visual changes touched exactly the intended lines. The final commit changed 44 files: +186 lines, −397.

## Using it

Skills live in your Claude Code skills folder. Clone this repo there:

```bash
git clone https://github.com/kellyruthw/brand-foundation.git ~/.claude/skills/brand-foundation
```

Then either:

- **Let it trigger:** ask Claude to start a build from a Figma file, set up design tokens, audit a design, or build a component. The skill loads when the request matches.
- **Call it directly:** type `/brand-foundation` at the start of a message.

It works on new and already-built projects. On a new one it starts with the design audit. On a built one it starts with the consistency pass, keeps the project's own token names, and adds only what the foundation is actually missing.

## About

Built by **Kelly Warpechowski**, design engineer at [Heartcore Studio](https://heartcore.studio). The process comes from how I build client sites: design system first, drift checks at every step.

## License

[MIT](LICENSE)
