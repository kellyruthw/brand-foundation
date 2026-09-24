# Review artifact

After the foundation is built, generate one self-contained HTML file
(`foundation-review.html`) so Kelly can check everything at a glance. Anything
that "sticks out" here would have stuck out across the whole site.

Use the real values, never a hand-typed copy. Sass variables produce no CSS
on their own, so pull them out with a small Sass file: use `sass:meta`'s
`module-variables()` on the variables module, output each one as a CSS
custom property, compile it, and build the page from that. Render the type
styles by actually applying each mixin to a sample class in the same compile.
Publish the page as an artifact when the session can; otherwise save it
outside the project's source folders and give Kelly the path.

## Sections, in order

1. **Colors** — one swatch per variable. Show the raw palette first, then
   semantic aliases (text, bg, border) grouped by role. Each swatch shows the
   variable name and hex. Put near-duplicates side by side so they're easy to
   compare.

2. **Type styles** — one row per mixin (`display-2xl`, `body`, `label`, …).
   Render the same sample line twice: once at the token's min size and once at
   its max, with the family, weight, line-height, and letter-spacing printed
   underneath. Include any nested variants a mixin defines (e.g. an italic `span`).

3. **Type roles** — h1 through h6 and body as they'll actually render, so the
   hierarchy reads as a whole.

4. **Spacing** — one horizontal bar per spacing/padding/gap variable, drawn to
   scale, labeled with name and value. Uneven steps show up immediately.

5. **Breakpoints** — a labeled ruler, with each header-height token marked at
   the breakpoint where it applies.

6. **Gradients** — one block per gradient mixin.

7. **Surfaces** — each `.bg-*` class with a heading, paragraph, and link on it
   so text contrast can be checked.

8. **Status tones** — success, warning, error, and info side by side, each
   rendered as a real banner and badge (border + soft fill + text). Print the
   text-on-fill contrast ratio under each; flag anything under 4.5:1. Two tones
   that read alike are a problem even if the hexes differ.

9. **Shadows** — one white card per level (sm / md / lg) on the page
   background, labeled with the level and the value.

10. **Radii** — one box per radius token at a size typical of what uses it (a
    16px checkbox for xs, a button for sm, a card for md), so it's obvious
    whether the corners scale with the element.

11. **Spacing grid** — state the base unit, and add a warning for any spacing
    value that isn't a multiple of it.

12. **Warnings** — a list generated from the foundation itself:
   - any magenta (`#ff00ff`) swatch or "TODO …" font: a template key that
     was never filled
   - variables with identical values
   - size tokens referenced by mixins but missing from `$sizes`
   - semantic aliases that point to a color no longer in the palette
   - any `TODO: confirm with design`

Keep it plain. No framework, no external requests. It's a proof sheet, not a
style guide site.
