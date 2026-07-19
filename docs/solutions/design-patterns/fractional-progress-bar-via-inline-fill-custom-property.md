---
title: Turn an existing element into a fractional progress bar with one inline `--fill` custom property — no fill child element
module: sooks-saga-scroll
date: 2026-07-19
problem_type: design_pattern
component: frontend_stimulus
severity: low
resolution_type: code_fix
applies_when:
  - "You want an existing element (a header, a label bar, a row) to read as a progress/completion bar filled to a runtime fraction, without restructuring its markup"
  - "The fill percent is knowable at render time from data you already compute (a done/total, a score, an elapsed/total)"
  - "You'd otherwise add a nested fill `<div>` sized to width:X% just to show the fill"
tags:
  - css
  - custom-properties
  - progress-bar
  - gradient
  - render-time
  - no-extra-dom
---

# Turn an existing element into a fractional progress bar with one inline `--fill` custom property — no fill child element

## Context

The collapsed "COMPLETED" rollup bar was a flat label. It needed to read as a
*completion progress bar* filled to how much of the saga is done — the same
fraction the saga header's Complete bar shows. The obvious implementation adds a
nested fill `<div style="width:{pct}%">` inside the header and layers the label
over it. That means new markup, a positioning context, and z-index juggling for
an element that already exists and already holds its own text.

## Guidance

**Set a single `--fill` custom property inline at render time and drive the
element's own `background` gradient (and a `::before` edge) from it.** No fill
child, no width math on a nested box — the element paints its own fill.

The render computes the percent from data it already has and injects one value:

```js
// renderCompletedRollup — same source the header Complete bar uses
const st = sagaStatus(char, saga);              // { done, total, ... }
const pct = st.total > 0 ? Math.round((st.done / st.total) * 100) : 0;
`<div class="completed-rollup-header" style="--fill: ${pct}%"> … </div>`
```

The CSS reads `--fill` as a gradient stop: fill colors up to `var(--fill)`, then a
track color from `var(--fill)` to 100%. A hard stop (same position twice) gives a
crisp fill/track boundary; a `::before` rule at `left: var(--fill)` draws the
leading edge:

```css
.completed-rollup-header {
  position: relative;
  overflow: hidden;
  --fill: 0%;                       /* fallback if the inline value is absent */
  background: linear-gradient(90deg,
    var(--fill-a) 0%,
    var(--fill-b) calc(var(--fill) - 12%),   /* fill gradient */
    var(--fill-c) var(--fill),
    var(--track)  var(--fill),               /* hard boundary: track starts here */
    var(--track)  100%);
}
/* crisp leading edge at the fill boundary */
.completed-rollup-header::before {
  content: ""; position: absolute; top: 2px; bottom: 2px;
  left: var(--fill); width: 2px; background: var(--edge);
}
```

Two guards worth building in:

- **Provide a `--fill` fallback** in the rule (`--fill: 0%`) so an element that
  renders without the inline value still paints sanely (empty bar) rather than
  with an invalid gradient.
- **`calc(var(--fill) - N%)` clamps at 0** — for small fills the offset stop goes
  negative and the browser clamps it to the prior stop, which renders fine. For a
  100% fill the track segment is zero-width (full bar). Both extremes are safe;
  no per-value branching needed.

## Why This Matters

The fill state is *data*, and CSS custom properties are the seam for passing a
runtime value into a static stylesheet. Modeling the fill as one inline `--fill`:

- **Removes DOM** — no wrapper/fill child, no extra positioning context, no
  z-index stack to keep the label above the fill. The element reuses its own box.
- **Keeps the styling in CSS** — the gradient, edge, and colors live in the
  stylesheet; the render injects only a number. You can restyle the bar (colors,
  edge, glow) without touching JS.
- **Single source of truth for the fraction** — compute it from the same function
  the sibling bar uses (`sagaStatus`), so the two bars can never disagree.

## When to Apply

- Any existing element you want to read as a fill/progress bar where the percent
  is known at render time (completion, score, capacity, elapsed).
- Prefer the nested-fill-`<div>` approach instead when the fill must **animate on
  a timeline** (transition width over time), needs its own overflow/clip
  independent of the container, or must be a real focusable/measurable sub-element
  — a background gradient can't be transitioned as smoothly and isn't a box.

## Examples

- **Shipped in build `07192026.2`** (unit U7): the collapsed COMPLETED rollup
  header became a green→gold completion bar filled to `sagaStatus().done/total`
  via `style="--fill: ${pct}%"`, with a track for the remainder and a `::before`
  leading edge — no fill child added. Live file:
  `sooks-saga-scroll-07192026-2.html` (`renderCompletedRollup` sets `--fill`;
  `.completed-rollup-header` + `::before` consume it).
- **Related:** the CSS-cascade discipline docs in this project —
  `docs/solutions/design-patterns/important-display-defeats-container-fold-toggle.md`
  (keep structural CSS at the right specificity / gate on render-time state) is
  the sibling "let render-time state drive CSS, don't force it" learning.
