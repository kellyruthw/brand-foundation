# Design audit checklist

Go artboard by artboard first, then step back and look at the whole set.

## Per artboard

**Color**
- List every fill, stroke, and text color used, as hex.
- Group values that are within a few steps of each other (same hue, nearly
  the same lightness). These are the "similar but inconsistent" candidates.
- Note where a color is used: text, surface, border, accent. The same hex
  doing two jobs is fine; two hexes doing one job is a flag.

**Type**
- Every distinct combination of family, size, weight, line-height,
  letter-spacing, and case.
- Sizes that differ by 1–2px across screens are usually one style, not two.
- Note which styles are clearly headings, body, labels, and which are
  unclear.

**Sizing and spacing**
- Section padding at each breakpoint shown.
- Gaps between repeated elements (card grids, lists).
- Container widths and column counts.
- Button and input padding, border radius, border width.
- Corner radius per element size: do small things get smaller corners than
  big ones, consistently?
- Shadows: list each distinct one. Is there a clear sm / md / lg, or a pile of
  one-offs? What color are they tinted with?

**Components**
- Anything that appears more than once, across screens or within one.
- Variants: same component with a different color, size, or icon.
- Near-duplicates: two things that look like the same component but differ
  in one measurement. These are either a variant or an inconsistency; decide
  which and flag.

## Whole-system pass

- Does a type scale exist? (Sizes step in a pattern, or are they arbitrary?)
- Does a spacing rhythm exist? What base unit does it step on (4px? 5px?)?
  Confirm the base with the designer rather than inferring it, and list the
  values that fall off it.
- Status colors: which colors mean success, warning, error, and info? Does
  any color do two of those jobs (e.g. the same orange for "good" and
  "careful")? Check each status text color against its fill for 4.5:1.
- How many distinct colors are there really, once near-duplicates merge?
- Which breakpoints does the design actually show, and which does the build
  need that it doesn't?
- Are hover, focus, active, disabled, error, and empty states designed? List
  what's missing.

## Report format

Use this structure. Keep it scannable; Kelly reads it before the designer
meeting.

```
# Audit: [Project name]

## Confirmed patterns
Things that clearly repeat and become tokens. One line each.
- Type scale: 14 / 16 / 18 / 22 / 28 / 50 / 78 px (fluid between 550–1728vw)
- Section padding: 60 mobile / 100 tablet / 150 laptop / 180 desktop

## Inconsistencies
Each one: what, where, recommended resolution.
- Navy: #22336C on Home, About; #22346C on Contact hero. Recommend #22336C
  (used 11×, the other 1×).
- H3: 28px on most screens, 26px on Blog list. Recommend 28px.

## Gaps
States or breakpoints the build needs that aren't designed.
- No focus state on inputs
- Nothing shown between 768 and 1024px

## Questions for design
Only what's actually ambiguous. One line each, answerable in one line.
1. #22336C vs #22346C — intentional?
2. Is the 26px blog heading a distinct style or should it be h3?
3. Should the card grid gap match the 60px section gutter, or stay 40px?
4. What's the spacing base unit: 4px or 5px?
5. Status colors: success / warning / error / info = ?
```
