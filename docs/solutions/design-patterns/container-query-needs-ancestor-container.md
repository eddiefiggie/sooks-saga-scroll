---
title: A container query needs an ancestor container — never target the element that declares container-type
module: sooks-saga-scroll
date: 2026-07-18
problem_type: design_pattern
component: frontend_stimulus
severity: medium
applies_when:
  - "Using CSS container queries (`container-type` + `@container`) for per-element responsive layout"
  - "An `@container` rule's selector is the same element that declares `container-type`"
  - "A responsive layout silently never changes at any width, yet the CSS is valid and passes syntax checks"
tags:
  - css
  - container-queries
  - responsive
  - layout
  - gotcha
  - containment
---

# A container query needs an ancestor container — never target the element that declares container-type

> Framework-agnostic CSS gotcha (vanilla CSS here); the `component: frontend_stimulus` tag is only the schema's nearest frontend bucket — no Stimulus involved.

## Context

While building a responsive "roll-over" tier system for the saga container headers and quest rows, the goal was for each row to restack **based on its own width** (so all same-width rows roll in lockstep). The natural tool is CSS container queries. The first cut wrote it like this:

```css
.saga-header {
  container-type: inline-size;   /* make this row a query container */
  display: grid;
  grid-template-areas: "t1 t2";
}
@container (max-width: 470px) {
  .saga-header {                  /* ...and restack it when narrow */
    grid-template-areas: "t1" "t2";
  }
}
```

It looked correct, passed the syntax check (`jsc` / any CSS linter is happy — it is valid CSS), and shipped. But **nothing ever rolled at any width** — the narrow layout was dead code. Because there is no runtime error and the syntax is valid, only a real-browser visual pass or a code reviewer catches it. Here, a code review caught it one build after it shipped.

## Guidance

**An `@container` rule resolves against the nearest *ancestor* query container of the element being styled. An element that declares `container-type` is a container for its *descendants* — it is never its own container.** So a rule that both (a) declares `container-type` and (b) is the selector targeted inside an `@container` block for that same container can never match: the styled element has no ancestor container (unless one happens to exist higher up), so the rule is inert.

Put `container-type` on a **wrapper/ancestor** of the element whose rules live inside `@container`:

```css
/* container-type goes on the PARENT... */
.saga { container-type: inline-size; }          /* the card, parent of .saga-header */

/* ...and the @container rules target the DESCENDANT */
.saga-header { display: grid; grid-template-areas: "t1 t2"; }
@container (max-width: 470px) {
  .saga-header { grid-template-areas: "t1" "t2"; }   /* now this matches */
}
```

Pick a wrapper whose width tracks the element you actually want to measure (here the card width ≈ the header width, so it's the right measuring context).

**One safety check before choosing the wrapper:** `container-type: inline-size` also applies `contain: layout style inline-size`, which makes the wrapper a containing block for `position: fixed`/`absolute` descendants and a new stacking context. Confirm nothing inside the wrapper relies on a *higher* containing block. In this app the only `position: fixed` elements (the autocomplete toast, the corner clusters) live outside the cards, and the card was already `position: relative`, so adding containment changed no coordinate origin.

## Why This Matters

Container queries fail **silently and invisibly**: valid CSS, no console error, no lint/syntax warning — the responsive behavior simply never happens. That makes this exact mistake (container-type on the queried element) especially costly: it survives every automated gate this project has (`jsc` syntax check, static grep) and only surfaces when a human resizes the window or a reviewer reasons through the cascade. A whole build shipped "done" with a completely non-functional feature. Knowing the ancestor rule up front, and treating "responsive layout that never changes" as a prime suspect for it, turns a dead-build round-trip into a one-line fix.

## When to Apply

- Any time you reach for container queries to make a component respond to its own (not the viewport's) width.
- When a container-query-driven layout "does nothing" — check first whether `container-type` sits on the very element the `@container` rules target; if so, move it to a wrapper.
- As a review checklist item on any diff that adds `container-type`: verify the `@container` rules target *descendants* of it, not the element itself.

Pairs with [matchmedia-reparent-fixed-clusters](./matchmedia-reparent-fixed-clusters.md) — both are responsive-layout mechanisms in this app; that one moves fixed corner clusters into flow at a viewport breakpoint, this one restacks rows at each row's own width. Also relevant: container-query correctness can't be caught by [the jsc syntax check](../developer-experience/verify-single-file-html-without-node.md) — it needs a visual or reviewer pass.

## Examples

**Broken (container-type on the queried element — narrow rule never fires):**

```css
.quest { container-type: inline-size; display: grid; grid-template-areas: "tog ctrl name"; }
@container (max-width: 560px) {
  .quest { grid-template-areas: "tog name" "ctrl ctrl"; }   /* DEAD — .quest has no ancestor container */
}
```

**Fixed (container-type on the parent; the queried element is now a descendant):**

```css
.quest-block { container-type: inline-size; }               /* parent of .quest */
.quest { display: grid; grid-template-areas: "tog ctrl name"; }
@container (max-width: 560px) {
  .quest { grid-template-areas: "tog name" "ctrl ctrl"; }   /* now matches on .quest-block's width */
}
```

(Shipped in `sooks-saga-scroll-07182026-5.html`, fixing the dead `.4` build.)
