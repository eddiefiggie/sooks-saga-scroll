---
title: Reflow fixed corner clusters into one column via a matchMedia reparent
module: sooks-saga-scroll
date: 2026-07-18
problem_type: design_pattern
component: frontend_stimulus
severity: medium
applies_when:
  - "position:fixed elements in different DOM parents must become one coherent ordered column at narrow widths"
  - "Pure-CSS order:/flex can't unify them because they don't share a flex/grid parent"
  - "Desktop layout must stay untouched while narrow layout is fully reordered"
tags:
  - responsive
  - matchmedia
  - dom-reparent
  - reflow
  - css-has
  - layout
  - vanilla-js
---

# Reflow fixed corner clusters into one column via a matchMedia reparent

> Framework-agnostic pattern (vanilla JS + CSS); the `component: frontend_stimulus` tag is only the schema's nearest frontend bucket — no Stimulus is involved.

## Context

The app has two `position:fixed` corner clusters — `.data-corner` (top-right: character picker + LFM alert cards) and `.data-corner.left` (top-left: server-pop + guild cards) — plus a Filters panel that lives in normal document flow, deep inside `<header>`. At desktop width the fixed clusters float in the corners; below ~1179px the CSS switches them to `position:static` and they fell into independent centered wrapping rows with **no defined order**, reading as a scrambled jumble.

The goal was one coherent narrow-width priority column: **Filters → Characters → LFM alerts → Server**. The blocker: pure CSS `order:` can only reorder siblings within a shared flex/grid parent, but the three sources live in **different DOM parents** — the two clusters are direct `<body>` children while Filters is nested inside `<header>` inside `.scroll`. Nothing short of moving nodes can interleave them.

## Guidance

Use a `matchMedia` handler that **moves the cluster nodes** into a dedicated flow container when narrow and **restores them to their exact original positions** when wide. Move nodes; never clone.

```js
(function initNarrowStack() {
  const stack = document.getElementById("narrowStack");        // empty div placed right after the Filters panel
  const right = document.querySelector(".data-corner:not(.left)");
  const left  = document.querySelector(".data-corner.left");
  if (!stack || !right || !left) return;                        // fail gracefully; nothing after depends on this

  const mq = window.matchMedia("(max-width: 1179px)");          // MUST match the CSS breakpoint exactly
  // Pin each cluster's original DOM home so restore is precise even though
  // the clusters share <body> with other elements.
  const rightHome = document.createComment("data-corner home");
  const leftHome  = document.createComment("data-corner.left home");
  right.parentNode.insertBefore(rightHome, right);
  left.parentNode.insertBefore(leftHome, left);

  function apply(narrow) {
    if (narrow) {
      if (right.parentNode !== stack) stack.appendChild(right); // right (Characters+LFM) above left (Server)
      if (left.parentNode  !== stack) stack.appendChild(left);
    } else {
      if (right.parentNode !== rightHome.parentNode) rightHome.parentNode.insertBefore(right, rightHome.nextSibling);
      if (left.parentNode  !== leftHome.parentNode)  leftHome.parentNode.insertBefore(left, leftHome.nextSibling);
    }
  }
  apply(mq.matches);
  if (mq.addEventListener) mq.addEventListener("change", e => apply(e.matches));
  else if (mq.addListener) mq.addListener(e => apply(e.matches)); // older Safari
})();
```

Four correctness rules make this safe:

1. **Move, don't recreate.** `appendChild`/`insertBefore` relocate the existing node, so event bindings, live-refresh timers, and every descendant `id` survive. Code that looks children up by `id` (e.g. a 15s refresh tick repainting `#serverPop`/`#guildCorner`) keeps working unchanged.
2. **Breakpoint parity.** The JS `matchMedia` query must equal the CSS `@media` breakpoint where the clusters flip `position:fixed → static`. A mismatch leaves a width band where a reparented cluster is still fixed-positioned (floating over content) or an in-flow cluster is still styled for the corner.
3. **Idempotent `apply()`.** The `parentNode !==` guards make redundant events (and repeated wide↔narrow toggling) no-ops, so nodes never duplicate or leak.
4. **Comment-marker restore.** Insert a comment node immediately before each cluster at init; restore with `insertBefore(node, marker.nextSibling)`. Markers are never moved into the stack, so they can't be orphaned, and the desktop DOM returns byte-identical across any number of cycles.

**Gotcha — dangling group labels.** If the narrow column adds a section divider/label (a `::before` heading) to a reparented cluster, gate it on the cluster actually having visible content, or it renders over empty space when the cluster's panels are hidden:

```css
/* Only show the "Server" divider+label when a panel is non-empty (panels hide via :empty{display:none}) */
#narrowStack .data-corner.left:has(.corner-panel:not(:empty))::before { content: "Server"; }
#narrowStack .data-corner.left:has(.corner-panel:not(:empty))       { border-top: 1px solid …; }
```

A `::before` lives on the always-present parent, so it renders regardless of child visibility. This exact dangling-label case was the one bug an adversarial code review caught in the first build.

## Why This Matters

`position:fixed` makes an element's DOM location irrelevant to its desktop rendering — which is precisely what lets you relocate it in the DOM for the narrow case without disturbing the desktop layout. The reparent gives full control over cross-parent ordering that CSS alone cannot express, while the marker-based restore guarantees the desktop view stays exactly as it was. Cloning instead of moving would silently break anything that holds a reference to the original node or repaints it by `id`.

## When to Apply

- A responsive layout must merge elements from **different DOM parents** into one ordered sequence at a breakpoint, and CSS `order:` can't reach across the parents.
- The wide/desktop layout must be preserved byte-for-byte while the narrow layout is fully restructured.
- The moved subtrees carry live state (timers, bindings, ids) that must survive the move — favor node relocation over re-render.

## Examples

- **Placement:** `#narrowStack` is an empty `<div>` sitting immediately after the Filters panel and before `<main>`. It's empty (and invisible) at desktop width; the handler fills it only when narrow, so the column reads Filters → (reparented clusters) → saga list.
- **Ordering within the stack:** append the right cluster first, then the left, so Characters + LFM land above Server — the two clusters' internal DOM order already matched the desired sub-order, so no per-panel moves were needed.
- **Verification without a browser:** parse-check the handler with `jsc` (see [verify-single-file-html-without-node](../developer-experience/verify-single-file-html-without-node.md)); reason through the toggle cycles for idempotency and marker integrity; defer the visual pass to the user.
