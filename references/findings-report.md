# Findings report

After a Phase 5 consistency pass, build one self-contained HTML page
(`findings-report.html`) so Kelly can *see* each issue instead of reading a
list of hex codes and line numbers. The chat keeps a short text summary; the
page is where the detail lives.

The rule for every finding: **show the problem, don't describe it.** If a
finding is about a color, render the color. If it's about size, render the
size. A reader should grasp most findings without reading the sentence under
them.

## Page structure, in order

1. **Header.** Project name, date, and one line on scope ("52 SCSS modules +
   the abstracts"). Then a count per group, as three tiles:
   **Safe fixes** · **Visual changes** · **Needs a decision**. The tiles jump
   to their sections.

2. **Needs a decision.** First, because only Kelly can unblock these. One
   card per question:
   - the question in one line
   - the options, rendered (two swatches, two sample sizes)
   - your recommendation and a one-line reason
   - where it's used (file:line list, collapsed if long)

3. **Visual changes.** One card per fix that alters rendering, with
   **before → after** rendered side by side (a 10px gap as a bar next to the
   12px bar; a 2px corner next to the 4px corner, on a box the size of the real
   element).

4. **Safe fixes.** Grouped by type (font sizes that skip tokens, raw radii,
   dead variables, stale comments …). Each type is one collapsible group with a
   count, and each row gives file:line, the current value and the token it
   becomes. There's no before/after preview, since nothing changes visually;
   say so once at the top of the section.

5. **Verification.** Fill this in after the fixes are applied: modules
   compiled, and the result of the before/after comparison of compiled CSS.
   For example: "52 modules, 0 errors. Safe fixes: identical CSS. Visual
   changes: 7 lines, all listed above."

## How to show each kind of finding

| Finding | Render it as |
|---|---|
| Near-duplicate colors | Two large swatches touching edge to edge, each labeled with variable name + hex |
| Wrong/off-scale font size | The same sample line at the current size and at the token size, sizes labeled |
| Off-grid spacing | Horizontal bars drawn to scale: current value vs the nearest grid values |
| Radius | A box at the element's real size, rendered with the current and proposed corner |
| Shadow | A white card on the page background, rendered with each shadow being compared |
| Contrast failure | The real text color on the real background, at the real size, with the ratio and pass/fail against 4.5:1 |
| Status tones that read alike | The tones rendered as real banners/badges side by side |
| Duplicate or sibling components | The two class names and files side by side, with the properties that differ highlighted |
| Dead code | File path and what it was; no preview |

## Every card carries

- a **file:line** reference in monospace. It's clickable if the page can link
  into the editor; otherwise it's easy to copy, since Kelly often makes the
  fix herself.
- the **category** as a small label.
- a **checkbox** so Kelly can tick off what she's handled. Store the ticks in
  the viewer's browser only (wrapped in try/catch); they're a personal
  convenience, not shared state.

## Keep it plain

- Self-contained: no framework, no external requests except web fonts.
- Render swatches and samples with the project's **real** values, read from
  its variables file, not retyped by hand. A typo in the report would be a
  finding about the report.
- Readable in light and dark mode, and usable at phone width.
- Don't restyle it with the project's brand. It's a working document, and
  brand colors would blur into the swatches being compared.
