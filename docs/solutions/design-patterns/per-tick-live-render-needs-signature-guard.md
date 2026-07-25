---
title: "Per-tick live renders need a signature guard: skip the DOM rebuild when the inputs are unchanged"
module: sooks-saga-scroll
date: 2026-07-25
problem_type: design_pattern
component: frontend_stimulus
severity: medium
resolution_type: code_fix
applies_when:
  - "A UI surface repaints on a recurring poll/tick (here the 15s DDO Audit live refresh) and rebuilds its DOM every tick even when nothing changed"
  - "Re-rendering re-binds event listeners each tick, risking a lost click when a tick lands between hover and click"
  - "You are adding a new live-refreshed surface (medallions, guild roster, LFM index, banners) and want it to match the existing no-churn pattern"
tags:
  - render
  - live-poll
  - signature-guard
  - dom
  - event-listeners
  - performance
  - gotcha
---

# Per-tick live renders need a signature guard

> Framework-agnostic pattern (vanilla JS here); `component: frontend_stimulus` is only the schema's nearest frontend bucket — no Stimulus involved. Companion: [gpu-cheap-continuous-css-animations](./gpu-cheap-continuous-css-animations.md) governs *how* to animate a live surface cheaply; this governs *when* to repaint it.

## Context

The app polls DDO Audit every 15s (`liveRefreshTick`) and repaints the live surfaces — character medallions, guild roster, LFM badges, and (as of build 07252026.2) the connect-switch banner. A surface that unconditionally rewrites its `innerHTML` every tick churns the DOM and re-binds its listeners **even when the underlying data has not changed**. Beyond wasted work, a rebuild that lands between a user's hover and click swaps the button node out and drops the click.

## Guidance

Guard the repaint with a **signature** of the rendered inputs. Compute a stable string from exactly the data the DOM reflects, store the last-rendered signature at module scope, and skip the rebuild when it is unchanged:

```js
let _sig = "";
function renderX(inputs) {
  const shown = derive(inputs);        // the exact data the DOM will reflect
  const sig = shown.join("\n");        // a separator that cannot appear in the data
  if (sig === _sig) return;            // unchanged since last render — no DOM churn
  _sig = sig;
  el.innerHTML = build(shown);         // rebuild + re-bind listeners only on real change
}
```

Two rules keep it correct:

- **The signature must capture everything the DOM reflects.** If a visible field changes but is not in the signature, the guard wrongly suppresses the repaint. Keep it to the *visible* data (not incidental state) so it does not over-fire either. Choose a `join` separator that cannot occur in the values (`"\n"` — names have no newlines); an empty-string join lets `["Ab","c"]` and `["A","bc"]` collide.
- **Any out-of-band clear must reset the signature.** A dismiss/close handler that sets `innerHTML = ""` without also resetting `_sig` can leave the surface stuck-empty — the next render sees the stale non-empty signature and skips. Route all clears through a helper that does both (`el.innerHTML = ""; _sig = "";`).

The codebase's canonical instance is `_lfmSig` for the live LFM index (repaint the saga list only when the LFM posting signature actually changes). The connect-switch banner added `_switchSig` (+ `_clearConnectSwitchBanner`) for the same reason after a code review caught the missing guard.

## Why This Matters

At a 15s tick an unguarded surface rebuilds ~240×/hour for no reason, re-binding listeners each time; the guild/LFM data usually changes far less often than the tick fires, so most rebuilds are pure waste. The subtler cost is correctness-adjacent: a rebuild between hover and click steals the click. The guard makes the repaint **event-driven** (on real change) rather than **clock-driven**.

## When to Apply

- Adding any surface that renders on the live tick (medallions, guild roster, LFM, raid card, banners).
- Any recurring-timer render whose inputs change less often than the timer fires.
- When a live surface's controls feel "flaky" under the poll — suspect a per-tick rebuild stealing the click.

## Examples

Reference implementation: the connect-switch banner in `sooks-saga-scroll-07252026-3.html` / `-4.html` (`renderConnectSwitch` + `_switchSig` + `_clearConnectSwitchBanner`), and `_lfmSig` in the guild/LFM path. Verify the guard in-browser by node identity — render, capture a child node, render again with identical inputs, assert it is the **same** node (guard skipped); then change the inputs and assert a **new** node:

```js
renderConnectSwitch(['Beta']);          const n1 = el.querySelector('.cs-banner');
renderConnectSwitch(['Beta']);          n1 === el.querySelector('.cs-banner');   // true — skipped
renderConnectSwitch(['Beta','Zeta']);   n1 !== el.querySelector('.cs-banner');   // true — rebuilt
```

Cross-reference the in-browser verification workflow: [verify-single-file-html-without-node](../developer-experience/verify-single-file-html-without-node.md).
