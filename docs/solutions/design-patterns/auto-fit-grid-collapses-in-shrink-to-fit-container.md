---
title: A repeat(auto-fit) grid collapses to one column inside a shrink-to-fit container — it needs a definite width to wrap
module: sooks-saga-scroll
date: 2026-07-19
problem_type: design_pattern
component: frontend_stimulus
severity: medium
applies_when:
  - "Using `grid-template-columns: repeat(auto-fit, <track>)` (or `auto-fill`) to wrap items into aligned columns"
  - "That grid is nested in a content-sized context — an `inline-flex` cluster, a flex item without `flex-grow`, or a `width: max-content`/`fit-content` box"
  - "The items stack in a single column at EVERY width instead of sitting in a row when there is room"
tags:
  - css
  - grid
  - auto-fit
  - flexbox
  - responsive
  - layout
  - gotcha
---

# A repeat(auto-fit) grid collapses to one column inside a shrink-to-fit container — it needs a definite width to wrap

> Framework-agnostic CSS gotcha (vanilla CSS here); the `component: frontend_stimulus` tag is only the schema's nearest frontend bucket — no Stimulus involved.

## Context

The per-quest control cluster holds three equal-width (140px) action pills — **First Time Reaper**, **Complete for Saga**, **Skip** — that should read as a horizontal **row on wide screens** and **stack into a single aligned column only when the row can no longer fit** as the screen narrows. `repeat(auto-fit, 140px)` looks like exactly the right tool: it wraps children into as many fixed-width columns as fit, keeping them column-aligned. The first cut wrapped the pills in a `.q-actions` span and wrote:

```css
.completion-controls { display: inline-flex; flex-wrap: wrap; }   /* the cluster: checkbox + .q-actions */
.q-actions {
  display: grid;
  grid-template-columns: repeat(auto-fit, 140px);   /* wrap the 3 pills into aligned 140px columns */
  gap: 6px;
}
```

It passed the syntax check and looked plausible. But the pills stacked into a **single vertical column at every width — including wide**, which is the opposite of the intent. The bug surfaced only when the user resized to a wide window and saw the pills stacked where they should have been a row.

## Guidance

**`repeat(auto-fit, <track>)` (and `auto-fill`) resolve the number of columns from the container's *definite* available inline size. In a shrink-to-fit / content-sized context, there is no width to fill, so auto-fit places the minimum — one column — at every width.** A grid collapses to content-based (intrinsic) sizing when its container is any of: an `inline-flex`/`inline-grid` child, a flex item **without** `flex-grow` (so it stays at its content width), a `width: max-content`/`fit-content` box, or an `auto`-sized grid track. In all of these the grid's preferred size under auto-fit is one track wide, so it never shows a second column no matter how wide the viewport is.

For "**horizontal row while it fits, aligned columns when it can't**," pick one:

- **(a) Give the grid a definite / filling width** so auto-fit has room to compute columns — e.g. make the container `display: flex` (block-level, fills its line) and the grid a `flex: 1 1 auto` item, or set an explicit `width`. Then `repeat(auto-fit, 140px)` genuinely wraps 3 → 2 → 1 as the width shrinks.
- **(b) Don't use auto-fit at all** — use a plain **flex row** (which naturally stays a single non-wrapping row while the items fit) and switch to a stacked **column at an explicit breakpoint** for when they can't. This is what shipped here, because the cluster is deliberately content-sized (the dashed box hugs the pills), so option (a) would have forced the box full-width.

The shipped fix:

```css
.q-actions { display: flex; gap: 6px; align-items: center; }   /* a ROW at wide (nowrap by default) */
@container (max-width: 490px) {                                /* only once a 3-pill row can't fit */
  .q-actions { flex-direction: column; align-items: flex-start; }   /* stack into an aligned column */
}
```

## Why This Matters

Like most CSS layout gotchas, this fails **silently**: valid CSS, no console error, no lint/syntax warning — the grid just collapses to one column and the "responsive" behavior is wrong at every width. Because auto-fit *looks* like the canonical "wrap into aligned tracks" idiom, it's easy to reach for without noticing the container it lands in has no definite width. The failure mode reads as "my items are always stacked" and the instinct is to fiddle with the breakpoint — but the breakpoint is irrelevant; the grid was never going to make a second column. Knowing that **auto-fit needs a definite container width** turns a confusing all-widths-stacked bug into a one-line container fix (or a switch to the flex-row + breakpoint pattern).

## When to Apply

- Any time you nest a `repeat(auto-fit, …)` / `auto-fill` grid inside an `inline-flex`/`inline-grid` box, a non-growing flex item, or a `fit-content`/`max-content` container.
- When a grid that "should wrap into columns" is stacked in a single column at all widths — suspect the container is shrink-to-fit before touching the breakpoint. Confirm by giving the container a temporary explicit `width` and seeing the columns appear.
- As a review checklist item on any diff adding `repeat(auto-fit …)`: verify the grid's container resolves to a definite inline size (fills its parent or has an explicit width), not intrinsic content size.

Pairs with [container-query-needs-ancestor-container](./container-query-needs-ancestor-container.md) — both are silent, syntax-valid responsive-layout failures where the CSS is "correct" but inert/wrong, caught only by a real-browser resize or a reviewer, not by [the single-file syntax check](../developer-experience/verify-single-file-html-without-node.md).

## Examples

**Broken (auto-fit inside an `inline-flex` cluster → one column at every width):**

```css
.completion-controls { display: inline-flex; }        /* content-sized: no definite width to fill */
.q-actions { display: grid; grid-template-columns: repeat(auto-fit, 140px); gap: 6px; }
/* → the 3 pills stack in ONE column at 1400px just as at 400px */
```

**Fixed (flex row that stacks only at the real collapse breakpoint):**

```css
.q-actions { display: flex; gap: 6px; align-items: center; }   /* row while it fits */
@container (max-width: 490px) {
  .q-actions { flex-direction: column; align-items: flex-start; }   /* aligned column when it can't */
}
```

Verified in-browser (localhost + Claude-in-Chrome): at 1200px the pills measure the same top-y (one row); below ~490px container width they share one left-x (aligned column). Shipped in `sooks-saga-scroll-07192026-5.html`.
