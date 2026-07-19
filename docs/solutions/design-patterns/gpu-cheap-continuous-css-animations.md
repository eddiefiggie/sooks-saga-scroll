---
title: Continuous CSS animations pin the CPU — gate them, then animate a GPU-composited property
module: sooks-saga-scroll
date: 2026-07-18
problem_type: design_pattern
component: frontend_stimulus
severity: medium
resolution_type: code_fix
applies_when:
  - "A page runs many continuous (infinite) CSS animations at once and the browser tab shows high CPU / fan spin even while idle"
  - "An animation targets `background-position`, `box-shadow`, `top/left/width`, `filter`, or another property that cannot be GPU-composited"
  - "A decorative animation (sheen, shimmer, pulse, glow) is applied to every element in a class rather than only the few that warrant it"
tags:
  - css
  - animation
  - performance
  - gpu-compositing
  - repaint
  - overflow
  - gotcha
---

# Continuous CSS animations pin the CPU — gate them, then animate a GPU-composited property

> Framework-agnostic CSS gotcha (vanilla CSS here); the `component: frontend_stimulus` tag is only the schema's nearest frontend bucket — no Stimulus involved. Sibling gotchas: [container-query-needs-ancestor-container](./container-query-needs-ancestor-container.md), [matchmedia-reparent-fixed-clusters](./matchmedia-reparent-fixed-clusters.md). Verification workflow: [verify-single-file-html-without-node](../developer-experience/verify-single-file-html-without-node.md).

## Context

Chrome ran hot whenever the app was open with a progressed character — noticeable fan spin, high CPU even with no interaction. The cause was decorative CSS animation at scale. Every progress bar with *any* progress carried a `has-progress` class whose `::after` overlay ran a slow illuminated "sheen":

```css
/* BEFORE — the CPU hog */
.saga-progress-bar.has-progress .saga-progress-fill::after {
  background: linear-gradient(115deg, transparent 32%, rgba(255,246,214,0.5) 50%, transparent 68%);
  background-size: 220% 100%;
  background-position: 220% 0;
}
@media (prefers-reduced-motion: no-preference) {
  .saga-progress-bar.has-progress .saga-progress-fill::after {
    animation: progressSheen 3.2s linear infinite;   /* animates background-position */
  }
}
```

Two things multiplied the cost: the animation was applied to **every** bar with progress (not just the interesting ones), and a later "twin bars" change gave each container **two** bars (reaper + completion). A character progressed across all 62 sagas therefore had **~121 sheens animating simultaneously**, and `background-position` **cannot be GPU-composited** — each frame forces a raster repaint of each element, ~60 times per second. The box-shadow "pulse" glows (`@keyframes` animating `box-shadow`) were the same class of cost, though they only fired on rarer live states.

## Guidance

Fixing this is two independent levers. Apply both.

### 1. Gate the animation to the few elements that warrant it

Most decorative animation is applied by a broad class (`has-progress`, `is-active`) when only a small subset actually earns the emphasis. Narrow the *animated* selector — keep the overlay element on everything, but only animate where it means something. Here, the shimmer became a **completion reward**: only complete bars animate; partial bars get a static version of the same shine.

```css
/* AFTER — one static overlay for all; animation only on complete bars */
.saga-progress-bar.has-progress .saga-progress-fill::after {
  /* fixed highlight band; resting = a static gold shine (no animation) */
  transform: translateX(0);
}
@media (prefers-reduced-motion: no-preference) {
  .saga-progress-bar.all-done .saga-progress-fill::after,
  .saga-progress-bar.reaper.sealed .saga-progress-fill::after {
    animation: progressSheen 3.2s linear infinite;
    will-change: transform;   /* only on the few animated elements, not all */
  }
}
```

This alone dropped the running-animation count from ~121 to a single-digit handful (only the complete bars). Note two states shared the "complete" concept but used different class names (`.all-done` for the completion bar, `.reaper.sealed` for the reaper bar) — enumerate every real completion state, don't assume one class covers all.

### 2. Animate a GPU-composited property instead of a repaint-forcing one

Only `transform` and `opacity` (and `filter` on some paths) are cheap to animate — the compositor moves an existing layer without repainting. `background-position`, `box-shadow`, `width/top/left`, and most other properties repaint every frame. Rewrite the effect to ride `transform`/`opacity`:

- **Sheen** (was `background-position`): make the overlay a fixed highlight band and animate `transform: translateX(...)` across the filled area. The parent's `overflow: hidden` clips the band as it sweeps off each end, so the loop is seamless. `from { translateX(-130%) } to { translateX(130%) }`.
- **Pulse glow** (was animating `box-shadow`): keep the resting `box-shadow` **static** (painted once, no per-frame cost) and deliver the "breathing" via a pseudo-element overlay whose `opacity` animates `0 → 1 → 0`. The overlay carries the peak glow; opacity is composited.

```css
/* pulse via opacity overlay, not animated box-shadow */
.el.hot::after {
  content: ""; position: absolute; inset: 0; border-radius: inherit;
  pointer-events: none; opacity: 0;
  box-shadow: 0 0 18px rgba(255,47,31,1);   /* peak glow, static on the overlay */
}
@media (prefers-reduced-motion: no-preference) {
  .el.hot::after { animation: pulse 1.4s ease-in-out infinite; }
}
@keyframes pulse { 0%,100% { opacity: 0 } 50% { opacity: 1 } }
```

### Gotcha: a child's OUTER box-shadow is clipped by an ancestor's `overflow: hidden`

When you move a glow onto a pseudo-element overlay, watch the containing box. An element's *own* outer `box-shadow` escapes its `overflow: hidden` (overflow clips descendants and content, not the element's own shadow) — but a **child** pseudo-element's outer box-shadow **is** clipped to the parent's rounded rectangle, collapsing the halo. This bit the two container-bar pulses, whose bars have `overflow: hidden`.

Two ways out, pick by whether the glow is outer or inner:

- **Outer glow** → anchor the opacity overlay on a **non-clipping ancestor** (a wrapper without `overflow: hidden`), not on a pseudo-element inside the clipped box.
- **Inner glow** (`inset` shadow) → an inset shadow renders *inside* the child's box, so it is clip-safe; a `::before`/`::after` on the clipped element works. Place it at a `z-index` above the fill but below any text (equal `z-index` resolves by DOM order — a pseudo before the text span, both at `z-index:1`, paints under the text).

Elements with no `overflow: hidden` (tags, table cells, list rows) can use a plain pseudo-element overlay with an outer glow — the clipping trap is specific to the clipped ancestor.

### Preserve each rule's reduced-motion contract exactly

Animations often carry a `prefers-reduced-motion` guard, and not always the same one. Most used `@media (prefers-reduced-motion: no-preference) { animation: … }` (animate only when motion is allowed). One used the **inverted** shape — animate unconditionally, then `@media (prefers-reduced-motion: reduce) { animation: none; box-shadow: <distinct resting glow> }`. When converting, keep each rule's own guard and its distinct reduced-motion resting value; don't normalize them to one pattern.

## Why This Matters

The dominant, always-on CPU cost here was **not** heavy JS or layout — it was ~121 continuous repaints per frame from a decorative gradient sweep. Continuous CSS animation is invisible in a syntax check and easy to add one class at a time until the aggregate pins a core. Gating cut the *count*; GPU-compositing made the survivors free. Measured before/after: **~121 running sheens → 8**, and the 8 survivors animate `transform`, not `background-position`, so they force **zero** raster repaints.

The general rule: **an animated property that isn't `transform`/`opacity` costs a repaint per frame per element** — audit the *count × property* product, not just "does it look smooth."

## When to Apply

- Any tab that idles hot with visible motion — decorative shimmer/pulse/glow applied broadly is the first suspect.
- Before adding a new continuous animation to a class that can match many elements: prefer a static treatment for the common case and animate only the exceptional state, and reach for `transform`/`opacity` first.
- Whenever you move a glow onto a pseudo-element: check the containing box for `overflow: hidden` and whether the glow is inner or outer.

## Examples

Measure it directly in the browser (no profiler needed) — inject a **count-sensitive fixture** (progress in every container, only a few complete) so the gate's effect is visible, then count running animations and assert the animated property:

```js
// count continuously-running instances of a specific animation
document.getAnimations()
  .filter(a => a.animationName === 'progressSheen' && a.playState === 'running')
  .length;                                  // BEFORE: ~121   AFTER: 8

// prove the survivors animate a GPU-composited property, not a repaint-forcing one
const a = document.getAnimations().find(x => x.animationName === 'progressSheen');
a.effect.getKeyframes().some(k => 'transform' in k);          // true
a.effect.getKeyframes().some(k => 'backgroundPosition' in k); // false

// confirm no data-scaling pulse still animates box-shadow anywhere
document.getAnimations()
  .filter(a => a.playState === 'running' &&
               a.effect.getKeyframes().some(k => 'boxShadow' in k))
  .map(a => a.animationName);               // should be [] (or only intentional singletons)
```

A "fully complete" fixture is the wrong test — every bar stays complete and keeps animating (as transforms), so the *count* doesn't move even though the fix works. Use a fixture where most elements are the common (now-static) case and only a few are the animated exception. See [verify-single-file-html-without-node](../developer-experience/verify-single-file-html-without-node.md) for the fixture-injection + screenshot workflow this builds on.

Reference implementation: build `sooks-saga-scroll-07182026-9.html` (sheen rules near the `.saga-progress-bar…::after` block; `lfmPulse` opacity overlays for the container-bar / tag / quest-row / first-time-reaper-cell pulses). Plan and review trail: `docs/plans/2026-07-18-003-perf-reduce-cpu-animation-load-plan.md`.
