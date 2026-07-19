---
title: A container's fold/toggle breaks when a child overrides `display` with `!important` — keep structural display at plain specificity
module: sooks-saga-scroll
date: 2026-07-19
problem_type: design_pattern
component: frontend_stimulus
severity: medium
resolution_type: code_fix
applies_when:
  - "A container hides/shows its children through a descendant selector (`.panel.folded .row { display: none }`) — a fold, collapse/expand, accordion, or tab switch"
  - "You also give a child its own `display` (grid/flex) and reach for `!important` to make it apply"
  - "A responsive grid reserves a track/column for an element that is only conditionally rendered"
tags:
  - css
  - specificity
  - important
  - cascade
  - grid
  - fold
  - gotcha
---

# A container's fold/toggle breaks when a child overrides `display` with `!important` — keep structural display at plain specificity

## Context

A filter panel folds shut via one rule: `.controls.folded .ctl-row { display: none }`
— the toggle adds `.folded` to the panel and every `.ctl-row` child collapses. A
later change turned one of those children into a grid and, out of habit, used
`!important` to make the grid layout stick:

```css
.ctl-row-top { display: grid !important; }   /* ← the trap */
```

Result: clicking "fold" collapsed *most* of the panel but the grid row stayed
visible. The bug is invisible in the un-folded state (everything looks right) and
only shows when the container is collapsed — easy to ship if you don't test the
folded state.

## Guidance

**Never `!important` a `display` that a container's show/hide rule needs to
override.** A toggle that hides children with `.container.state .child { display:
none }` is a *descendant selector* — specificity `(0,2,0)`. A plain child rule
`.child { display: grid }` is `(0,1,0)`, so the container's rule wins when the
state is active and the child collapses. Add `!important` to the child and you
invert that — `!important` beats specificity, so the child refuses to hide.

```css
/* Container owns show/hide via a descendant selector — (0,2,0) */
.controls.folded .ctl-row { display: none; }

/* WRONG — !important (0,1,0 + !important) defeats the fold; row stays visible */
.ctl-row-top { display: grid !important; }

/* RIGHT — plain specificity; the fold rule's (0,2,0) still wins when folded,
   and .ctl-row-top's grid applies whenever it's NOT folded */
.ctl-row-top { display: grid; }
```

The default reflex/grid layout of a foldable child must sit at **plain
specificity** so the container's higher-specificity (or `!important`-free) toggle
can still win. If you think you need `!important` on a structural `display`, that's
the signal to stop and check what else sets `display` on the same element.

### Corollary — reserve a conditional grid track only when its content exists

The same "conditional presence" discipline applies to grid tracks. A header grid
that always declares a second column reserves that column's `column-gap` even when
the element that would fill it is absent:

```css
/* Always 2 columns — when .hdr-t3 (the optional LFM) isn't rendered, the `auto`
   track collapses to 0 but the 14px column-gap between col1 and col2 still sits
   there, leaving dead space on the right. */
.saga-header {
  grid-template-columns: minmax(0,1fr) auto;
  grid-template-areas: "t1 t3" "t2 t3";
  column-gap: 14px;
}
```

Fix: gate the extra track on a class set at render time only when the optional
element is present, so the default (single-column) case reserves nothing:

```css
.saga-header { grid-template-columns: 1fr; grid-template-areas: "t1" "t2"; }
.saga-header.has-lfm {
  grid-template-columns: minmax(0,1fr) auto;
  grid-template-areas: "t1 t3" "t2 t3";
}
```

```js
// renderSagaCard — add the class only when the optional element renders
`<div class="saga-header${lfmSkullTag ? " has-lfm" : ""}" ...>`
```

## Why This Matters

Both traps share a root cause: **CSS reserves layout for what the stylesheet
declares, not for what the DOM actually contains right now.** A fold rule and a
grid track are both static declarations; whether they should apply depends on
runtime state (folded? LFM present?). The wrong instinct is to force the static
rule with `!important` or an always-on track; the right one is to let specificity
and a render-time class model the state.

The `!important` variant is especially costly because it fails *silently and
partially* — the un-folded state looks perfect, so it passes a casual glance and
only breaks in the collapsed state a reviewer might not exercise. It cost a full
in-browser review cycle to catch here.

## When to Apply

- Any fold/collapse/accordion/tab where a container class hides children. Before
  adding `!important` to a child's `display`, confirm no ancestor-state rule needs
  to override it.
- Any grid that declares a track for a conditionally-rendered element — reserve
  the track behind a render-time class instead of always-on.
- When a toggle "mostly works" but one element won't hide/show: grep for
  `!important` on `display` (or `visibility`/`height`) of that element first.

## Examples

- **The bug, as shipped in a draft and caught in review** (build `07192026.2`,
  filter panel): `.ctl-row-top { display: grid !important }` left the filter grid
  visible when the panel was folded; removing `!important` restored the fold
  because `.controls.folded .ctl-row { display: none }` (0,2,0) then out-specified
  the plain `.ctl-row-top` (0,1,0). Live file:
  `sooks-saga-scroll-07192026-2.html` (`.controls.folded .ctl-row` and
  `.ctl-row-top` rules).
- **Verification approach:** test the *collapsed* state, not just the open one —
  the failure mode is invisible when expanded. See
  `docs/solutions/developer-experience/verify-single-file-html-without-node.md`.
- **Related:** `docs/solutions/design-patterns/container-query-needs-ancestor-container.md`
  is the sibling CSS-cascade gotcha in this project (a container query needs an
  ancestor container); both are "the rule looks right but resolves against the
  wrong scope/specificity" traps.
