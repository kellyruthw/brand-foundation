---
name: brand-foundation
description: Kelly's process for turning a client's Figma file (or any brand design system) into a coded foundation and building a site on top of it without drift. Use this whenever starting a new site or app build from a Figma file, setting up variables/tokens/mixins for a project, auditing a design for color/type/spacing inconsistencies, or building components against an existing design system. Also use it mid-build whenever a new component is about to be created, a color or size value is about to be hardcoded, or something looks like a one-off — the reuse and drift checks in this skill apply at every step, not just setup.
---

# Brand Foundation

This skill encodes how Kelly builds sites: audit the design first, confirm the
truth with the designer, encode that truth once as variables and mixins, review
it at a glance, then build component by component while refusing to create
anything the foundation already covers.

The reason this order matters: an AI building a site without a foundation will
happily invent a fourth shade of grey, hardcode a font size, or build a second
card component that is 3px different from the first. Each of those is small.
Together they are why AI-built sites look generic and drift out of spec. The
foundation is how a brand's judgment gets into the code before any page exists,
and the reuse rules are how it stays there.

## Before you start

**Check whether a foundation already exists.** Look for a variables/tokens
file, mixins, and components that use them. If the project is already built,
don't start at Phase 1. Run the Phase 5 consistency pass first to see how far
the build has drifted. Then add whatever the foundation is missing, using the
Phase 3 checklist (status tones, shadows, radii, spacing function, motion).
Retrofitting in place beats rebuilding a foundation under a finished site.

**Ask the spacing base unit.** Every spacing value should be a multiple of one
base. That's often 4px, but some designers use 5px, and it's the designer's
call, not yours. Ask once per project. Encode it as `$space-base` with a
`space($n)` function, and treat any value off that grid as drift.

**Don't carry one project's decisions into another.** Rules Kelly sets for a
specific site stay with that site. For example, heartcore-books has "no black
anywhere", line-height 1 on buttons and fields, 12px button side padding, and
its own status colors. Reuse the *structures* this skill describes, but get
the values from each project's design, and ask when they aren't clear.

## The five phases

Work through these in order. Do not start Phase 4 until Phase 3 has been
reviewed by Kelly.

### Phase 1 — Audit the design

Before writing any code, read the Figma file (or exported screens, or brand
guidelines) the way a design engineer would: artboard by artboard, then as a
whole. Read `references/audit-checklist.md` and produce an audit report using
the format there.

The point of the audit is not to list everything. It is to find:

- **Patterns** — values that repeat and clearly want to be tokens (a spacing
  rhythm, a type scale, a small set of surface colors)
- **Inconsistencies** — values that are *almost* the same and probably should
  be identical: `#22336C` and `#22346C`, a 24px heading on one screen and 26px
  on another, a button with 16px padding here and 18px there
- **Gaps** — states or breakpoints the design doesn't show but the build needs

Never resolve an inconsistency by picking one silently. Flag it, say which
value you'd recommend and why, and put it on the question list for design.
Choosing on the designer's behalf is how a brand's intent gets quietly
overwritten.

### Phase 2 — Confirm with design

The audit ends with a short list of questions for the designer. Keep it to
what is actually ambiguous. Every question should be answerable in one line
("Use #22336C everywhere", "The 26px one was a mistake", "Yes, that's a
distinct third grey").

Wait for answers. The confirmed values become the official variables. If a
question can't be answered yet, use the most common value from the design,
mark the token with a `// TODO: confirm with design` comment, and move on.

### Phase 3 — Build the foundation

Read `references/variables.scss` and `references/mixins.scss` first. They are
the starting point for every project and show the shape a foundation should
take. Every key in them should exist in every project, but none of the values
are real: brand colors, font names, font sizes, spacing and radii all come
from the project's confirmed values in Phase 2. The placeholders are
deliberately loud: colors are magenta `#ff00ff`, fonts are named "TODO …", and
per-project numbers carry `// TODO: from design`. Any magenta or TODO still
left when Phase 3 ends is an unfilled key, so ask about it rather than
inventing a value. Add or remove keys (an extra weight, a fourth accent) to
match the design, but keep the structure.

Build in this order, because each layer depends on the one before it:

1. **Variables** — fonts, weights, line-heights, letter-spacing, size tokens
   (min/max px pairs for fluid type), semantic roles (which token an h2 uses),
   grid, colors (raw palette first, then semantic aliases like
   `$color-text-primary` that point at the palette), breakpoints, spacing
   (`$space-base` + `space()`), status tones, shadows, radii.
2. **Mixins** — fluid type, size-token/size-role, type styles, layout/grid,
   gradients, responsive helper, reduced-motion helpers. Every mixin reads from
   variables; none of them contain a raw hex or px value.
3. **Globals** — html/body defaults and surface classes (`.bg-*`) that
   components sit on.

Rules for this phase:

- One source of truth per value. If two variables hold the same number
  (`$padding-a: 150px` and `$padding-b: 150px`), either they mean the same
  thing and one should alias the other, or they mean different things and the
  values should differ. Flag it either way.
- Semantic names over descriptive ones at the point of use. Components use
  `$color-text-primary`, never `$color-navy-800`, so a rebrand is a one-line
  change.
- Every size token referenced by a mixin must exist in `$sizes`. A mixin that
  calls `size-token(card)` when `card` isn't in the map fails at compile time,
  or worse, silently when the map lookup returns null.
- Class names should match token names. A surface class called `.bg-cream`
  that applies `$bg-tertiary` (navy) is a leftover from another project and
  will confuse the next person. Rename or delete.
- **Name status tones by what they mean in the UI:** `$tone-success`,
  `$tone-warn`, `$tone-error`, `$tone-info`. Never use "positive/negative" or
  the color's name. Each tone gets three values: a border/accent color, a soft
  fill, and a text color. Which color means what comes from the design, so ask.
  On heartcore-books, a "positive" token set to orange quietly made every
  success message orange, and that was invisible until warnings needed orange
  too.
- **Check contrast whenever you pick a text color**, especially text sitting
  on its own soft fill: body text needs 4.5:1. A brand accent used as text on
  its own tint often fails. If it does, keep the accent on the border and use a
  darker text color. Report the ratio when you propose a color.
- **Shadows go on an intensity scale:** `$shadow-sm` / `md` / `lg`. Each one
  is built from a `$shadow-color` variable, never a hardcoded
  `rgba(0,0,0,…)`, so the tint is one decision. Two floating elements (a
  popover and a tooltip, say) should share a level, not each invent their own.
- **Radius scales with the element:** a raw scale (`$radius-xs` / `sm` / `md`
  / `pill`) plus role aliases that components use (`$radius-control`,
  `$radius-card`). Small things get small corners. A nested box's radius should
  be its parent's radius minus the padding between them.
- **Every transition sits inside a reduced-motion mixin**, so people who turn
  motion off get instant state changes.

Then **generate the review artifact**: a single self-contained HTML page that
renders every color swatch (with its variable name and hex), every type style
at its min and max size, every spacing value as a bar, and every gradient. See
`references/review-artifact.md` for the layout. Kelly reviews this at a glance
to catch anything that "sticks out" before a single component exists. That
review is far cheaper than finding the same problem across forty components
later.

### Phase 4 — Build components, in order

Build in dependency order: primitives (buttons, links, labels, inputs) →
composites (cards, media blocks, nav) → sections → pages.

Before creating **any** new component, run the reuse check:

1. Does a component already exist that does this? Use it.
2. Does one exist that does 80% of this? Extend it with a variant or prop.
   Say what you added.
3. Is this truly new? Build it, and say so explicitly: "This is a new
   component. I checked X and Y and neither fits because ___." If it will be
   used elsewhere, build it reusable from the first pass.

The reuse check is the most important rule in this skill. Silent one-offs are
the failure mode. When Kelly says "wait, is this a one-off?", the honest answer
should already be in the build notes.

Also in this phase:

- No raw values in component styles. Colors, sizes, spacing, and breakpoints
  come from variables and mixins. If a value doesn't exist in the foundation,
  that is either a gap to add to the foundation (and flag) or a mistake in the
  build. It is never a reason to hardcode.
- Match the Figma, not the vibe. If a card has 32px padding in the design and
  the foundation has a 2rem gap token, use the token and note that it maps.
  If the design says 30px and nothing in the foundation is 30px, flag it
  instead of picking the closest.
- Keep the file structure legible. If a component renders differently in two
  places, stop and find out why before building more. A codebase that has
  drifted past what Kelly can follow is worse than a slower build.

### Phase 5 — Consistency pass

Before calling a page or milestone done, sweep for drift. Grep for each of
these, not just the obvious ones; the easy misses are marked.

- Any hex, px/rem font-size, or magic number in a component file
- Raw values passed as mixin *arguments*, e.g. `type-style($ff, $w, 1.25)`,
  which hides a line-height a `line-height:` grep won't find (easy miss)
- Pure black anywhere, including `rgba(0,0,0,…)` inside shadows (easy miss)
- Raw `box-shadow` and `border-radius` values that skip the scales
- Spacing off the project's base grid (a 9, 10, 17 on a 4px grid)
- Transitions or animations not wrapped in the reduced-motion mixin. Grep
  every `transition:` (easy miss: they hide inside long rule blocks)
- Any two components that look like siblings but were built separately, and
  any pattern block retyped identically in two modules
- Components nothing imports, and variables nothing references (an unused
  variable often means something was hardcoded instead)
- Two variables holding the same value, and near-duplicate hexes
- Comments that contradict the code they describe (a stale "24px/500" above a
  24px/400 style will mislead the next pass)
- Text colors that fail contrast on the surface they sit on
- Any `TODO: confirm with design` still open

Report findings with file and line, precise enough that Kelly can go straight
to each one. She often makes small pixel-level fixes herself because explaining
the tweak takes longer than doing it. Group them by what they need:

1. **Safe fixes**: no visual change (swapping a raw value for the token that
   holds the same value, deleting dead code, fixing comments). Offer to do
   these in one go.
2. **Visual changes**: fixes that alter rendering (snapping 10px to 12px,
   fixing a bug). List the before and after.
3. **Needs a decision**: design questions, one line each, with your
   recommendation.

**Verify by comparing compiled CSS, not by eyeballing.** Compile every module
before and after your edits (the way the bundler does, with the abstracts
prepended) and diff the output. A safe fix should produce identical CSS, and
a visual change should show exactly the lines you meant to change. This proves
the change without a full production build, which can starve Kelly's running
dev server.

## Working with Kelly

- She holds the Figma in her head while building. If she says something is
  off, assume she's right and look for the cause rather than defending the
  output.
- Small, precise changes over rewrites. If a fix is one line, make it one line.
- Say what you built and what you reused in each step. Reuse should be
  visible, not assumed.
- Ask when explaining is faster than guessing. Guess when guessing is cheap and
  reversible.

## Reference files

- `references/audit-checklist.md` — what to look for in Phase 1 and the report
  format
- `references/variables.scss` — the token template: every key a foundation
  needs, with placeholder values to replace per project
- `references/mixins.scss` — the matching mixins: fluid type, type styles,
  `space()`, `respond()`, motion, grid, and gradients. `respond()`,
  `size-token()` and `size-role()` fail the build on an unknown name, never
  silently.
- `references/globals.scss` and `references/main.scss` — how globals sit on
  top (one `.bg-*` surface class per `$bg-*` token, names matching) and how the
  entry file is organized
- `references/review-artifact.md` — layout for the at-a-glance HTML review page
