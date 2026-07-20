---
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
execution: code
product_contract_source: ce-plan-bootstrap
type: fix
title: "fix: Symmetric edge-pinned corner widgets that shrink (not grow) as the window narrows"
date: 2026-07-19
depth: lightweight
---

# fix: Symmetric edge-pinned corner widgets that shrink (not grow) as the window narrows

## Summary

The two floating header corner clusters (upper-right: Characters + reaper/raid LFM stack; upper-left: Server + Guild) currently **grow** from 200px to 320px and reflow into the center at a hard `1179px` breakpoint as the window narrows — the panels visibly enlarge and slide over the middle content. Replace the flat width + hard breakpoint with a single viewport-driven **shared corner width** that keeps both sides equal, holds 200px on wide screens, then **shrinks symmetrically** (never grows) so the clusters stay pinned at the left and right edges as long as geometrically possible, and only drop into the existing single-column narrow stack at the true collision point with the center parchment.

**Scope:** the narrowing/responsive behavior of the header corner clusters only. The wide-desktop fixed layout (≥ ~1620px) and the narrow single-column stack target are preserved; this plan changes *how and when* the transition between them happens.

---

## Problem Frame

`.corner-panel` (both clusters) and the center `.scroll` column are governed by three width regimes in the stylesheet, plus a JS reparent (`initNarrowStack`). Grounded in `sooks-saga-scroll-07192026-2.html`:

- **≥1621px** — `.data-corner` is `position:fixed` at `left:14px`/`right:14px`; `.corner-panel { width:200px }`; `.scroll { max-width:1150px; margin:0 auto }`. Symmetric, correct.
- **1180–1620px** — corners stay fixed at 200px; `.scroll { max-width: calc(100vw - 452px) }` cedes side gutters so cards never sit on the parchment. Still symmetric and edge-pinned — this is the behavior the user wants *extended*.
- **≤1179px** — `.corner-panel { width:320px; flex:0 1 320px }` (**the growth**), corners go `position:static; flex-wrap:wrap; justify-content:center` (they leave the edges and pile toward the center), and `initNarrowStack()` reparents both clusters into a 380px `#narrowStack` column above the scroll.

**Root cause of the reported symptom:** (1) an explicit `width:320px` applied *as the viewport shrinks* is literal growth-on-narrow; (2) the reflow is triggered by a fixed `1179px` media breakpoint rather than by actual collision with the center, so the clusters abandon the edges earlier than necessary. The desired behavior is equal-width, symmetric, edge-pinned corners that shrink together to persist as long as possible, reflowing only when they would truly collide with a readable center column.

**Related prior work:** `docs/plans/2026-07-18-001-feat-responsive-stack-and-header-bars-plan.md` introduced the current `#narrowStack` / fixed-corner responsive behavior. This plan refines its transition mechanics; the narrow single-column *target* it established is kept.

---

## Requirements

- **R1** — Left and right corner clusters are always **equal width** and symmetric about the center, at every viewport width.
- **R2** — Corner panels **never grow** as the viewport narrows; they hold their maximum width (200px) on wide screens and only ever shrink.
- **R3** — As the window narrows, the clusters **stay pinned at the left/right edges as long as possible**, shrinking symmetrically toward a readable floor, while the center `.scroll` yields side gutters and the corners never overlap the parchment.
- **R4** — When the corners can no longer shrink without colliding with a minimum readable center width, they reflow into the existing `#narrowStack` single column (right cluster above left cluster). No intermediate "grown + centered wrapping rows" state.
- **R5** — The CSS reflow breakpoint and the `initNarrowStack` JS breakpoint stay in sync (one source-of-truth value), so CSS reflow and DOM reparent happen at the same width.
- **R6** — Wide-desktop layout (≥ ~1620px) and the narrow single-column stack appearance are unchanged from today.

---

## Key Technical Decisions

- **KTD1 — A single shared `--corner-w` custom property drives both corner width and the center gutter reservation.** Define `--corner-w: clamp(<floor>, calc((100vw - <center-floor>)/2 - <inset+slack>), 200px)` once (on `:root`/`body`). `.corner-panel` uses `width: var(--corner-w)`; `.scroll` reserves `2 × var(--corner-w)` (+ insets) as gutter. Because both read the same clamp, left/right stay equal (R1) and the center yields exactly as the corners shrink (R3), with no possibility of the two formulas drifting out of sync. *(session-settled: user-directed — chose "shrink to persist even longer" over "constant width, reflow later": user wants the clusters edge-pinned to the smallest widths.)*
- **KTD2 — Shrink, floored, never grow.** The clamp's max is 200px (today's width) and its min is a readable floor (start at ~150px; tune in-browser). Between them the middle `vw` term shrinks the panels as the viewport narrows. The old `width:320px` growth rule is removed. Satisfies R2.
- **KTD3 — Reflow is driven by the collision point, expressed as one breakpoint constant shared by CSS and JS.** The corners can shrink to the floor; below the width where the floor no longer clears a minimum readable center, reflow. Pick a single breakpoint value (derived from the floor + center-floor math, ~1040px; tune in-browser), use it in the media queries **and** in `initNarrowStack`'s `matchMedia` string. Satisfies R4, R5.
- **KTD4 — Collapse the special `calc(100vw - 452px)` 1180–1620 band into one continuous `.scroll` formula** covering the whole fixed regime (above the reflow breakpoint), rather than a single hard-coded band. Removes the abrupt seam and makes the yield continuous.

---

## Reference: width regimes (after this change)

| Viewport | Corner width (each, equal) | Position | Center `.scroll` |
|---|---|---|---|
| ≥ ~1620px | 200px (clamp max) | fixed, pinned to edges | centered, ≤1150px |
| ~1620 → reflow bp | 200px → floor (shrinks) | fixed, pinned to edges | `100vw − 2·corner − insets` (yields) |
| ≤ reflow bp (~1040) | 100% of `#narrowStack` (~380px col) | static, reparented to `#narrowStack` | full-width column, corners stacked above |

Exact `floor`, `center-floor`, and reflow-breakpoint numbers are tuned in-browser in U3; the values above are the starting design targets.

---

## Implementation Units

### U1. Shared responsive corner width (equal, shrink-not-grow)

**Goal:** Replace the flat `width:200px` corner width with a single viewport-driven `--corner-w` clamp applied equally to both clusters, so panels hold 200px wide and shrink symmetrically toward a floor as the viewport narrows — never growing.

**Requirements:** R1, R2.

**Dependencies:** none.

**Files:** `sooks-saga-scroll-07192026-2.html` (source; changes land in the next date-stamped build copy per the garage build workflow) — the `<style>` block: `.corner-panel` rule (~line 3471) and a new `--corner-w` declaration on `:root`/`body`.

**Approach:**
- Define `--corner-w: clamp(<floor>, calc((100vw - <center-floor>)/2 - <inset+slack>), 200px);` once. Start values: floor `150px`, center-floor `688px`, inset+slack `26px` (14px edge inset + ~12px clearance, matching today's 452 vs 428 slack).
- Change `.corner-panel { width: 200px }` → `width: var(--corner-w)`. Both `.data-corner` and `.data-corner.left` inherit it identically, so L/R remain equal by construction.
- Leave `.data-corner`/`.data-corner.left` insets (`left:14px`/`right:14px`) unchanged.

**Patterns to follow:** the existing `.corner-panel` shared-width comment ("both cards share one width so they align") — this makes that guarantee viewport-wide.

**Test scenarios (in-browser, localhost + Claude-in-Chrome):**
- At 1620px: both panels measure 200px; left cluster width == right cluster width.
- At 1300px and 1180px: both panels equal and ≤200px (not >200); still pinned at edges.
- Sweeping 1620→1080px: panel width is monotonically non-increasing (never larger than at a wider width). Covers R2.
- Left and right panel widths are equal at every sampled width. Covers R1.

**Verification:** DevTools computed width of a `.corner-panel` in each cluster is identical and ≤200px across the sweep; no width exceeds 200px at any narrower width.

---

### U2. Symmetric center-gutter reservation tied to the same width

**Goal:** Make the center `.scroll` yield symmetric side gutters sized to the live `--corner-w`, replacing the hard-coded `calc(100vw - 452px)` band, so the shrinking corners stay edge-pinned and never overlap the parchment across the whole fixed regime.

**Requirements:** R3, KTD4.

**Dependencies:** U1 (consumes `--corner-w`).

**Files:** `sooks-saga-scroll-07192026-2.html` `<style>` — `.scroll` base (~line 248) and the `@media (min-width:1180px) and (max-width:1620px)` block (~line 4063).

**Approach:**
- Replace the single 1180–1620 band with one continuous rule for the fixed regime (above the reflow breakpoint): `.scroll { max-width: min(1150px, calc(100vw - 2*var(--corner-w) - <2·(inset+slack)>)); }` so the center never grows past 1150px on wide screens but yields continuously as corners shrink.
- Keep `margin: 0 auto` (symmetric centering) — combined with equal gutters this guarantees the corners clear the parchment equally on both sides.
- Confirm the reserved gutter ≥ corner footprint (`var(--corner-w) + 14px`) at every width so no overlap occurs.

**Patterns to follow:** the existing intent of the `calc(100vw - 452px)` band ("scroll cedes side gutters so cards never sit ON the parchment") — generalized to a continuous, corner-width-linked formula.

**Test scenarios (in-browser):**
- Sweep 1620→reflow-bp: the center parchment never underlaps either corner cluster (no visual overlap; a small equal gap remains on both sides). Covers R3.
- Left gap (viewport edge → parchment) equals right gap within a pixel or two at each sampled width. Covers R1/symmetry.
- At ≥1621px the center is centered at ≤1150px exactly as today. Covers R6.
- Horizontal scrollbar never appears on `body` during the sweep.

**Verification:** measured left/right gutters are symmetric and ≥ corner footprint at every sampled width; no `body` horizontal overflow.

---

### U3. Collision-driven reflow breakpoint; remove the grow/wrap regime; sync JS

**Goal:** Move the reflow from the fixed `1179px` breakpoint to the true collision point (one shared constant), delete the `width:320px` growth + centered-wrapping-rows regime, and update `initNarrowStack` so the DOM reparent fires at the same width as the CSS reflow.

**Requirements:** R4, R5, R6.

**Dependencies:** U1, U2.

**Files:** `sooks-saga-scroll-07192026-2.html` `<style>` — the `@media (max-width:1179px)` block (~line 4066, including the `.corner-panel { width:320px; flex:0 1 320px }` rule ~line 4082) and the `@media (max-width:480px)` block (~line 4144, keep); and the `<script>` `initNarrowStack` IIFE (`window.matchMedia("(max-width: 1179px)")`, ~line 16817).

**Approach:**
- Choose one **reflow breakpoint** value (design target ~1040px, tuned in U-level in-browser step) derived from `floor + center-floor` math so corners persist to the smallest safe width.
- Update the `@media (max-width:1179px)` query and any paired `min-width:1180px` boundary to the new breakpoint (`≤bp` for the stacked regime, `≥bp+1` for the fixed regime).
- **Remove** `.corner-panel { width:320px; flex:0 1 320px }`. Inside `#narrowStack`, `.corner-panel { width:100% }` already governs the stacked column, so panels fill the ~380px stack — no growth. If the JS-disabled fallback (static wrapping rows without `#narrowStack`) must still look sane, cap it with `width: min(320px, 100%)` rather than a flat 320px.
- Update `initNarrowStack`'s `matchMedia("(max-width: 1179px)")` to the same breakpoint constant. Consider a brief comment noting the value is duplicated in the CSS media queries and must be kept in sync (R5).
- Leave the `#narrowStack` column rules, the `:has(.corner-panel:not(:empty))` Server divider, and the `@media (max-width:480px)` full-width rule unchanged.

**Execution note:** This is a visual/layout change with a shared CSS↔JS constant; prefer runtime in-browser verification over unit tests, and tune the floor / center-floor / breakpoint numbers against the live sweep before finalizing.

**Test scenarios (in-browser):**
- Just above the reflow breakpoint: both clusters fixed and edge-pinned at the corner floor width; center still readable.
- Crossing the breakpoint downward: CSS reflow and `initNarrowStack` reparent happen at the **same** width (no intermediate frame where corners are grown, centered, and overlapping). Covers R4, R5.
- At the stacked width: right cluster (Characters + LFM) renders above left cluster (Server) in the ~380px `#narrowStack` column, exactly as today. Covers R6.
- At ≤480px: panels are full-width as today. Covers R6.
- No `.corner-panel` anywhere renders at 320px at any width. Covers R2/R4.

**Verification:** DOM inspection confirms clusters reparent into `#narrowStack` at the same width the CSS switches to the stacked layout; the 320px width is absent from computed styles at all widths; narrow stack and ≤480px appearance match the pre-change build.

---

## Definition of Done

- Across a continuous resize sweep from ≥1621px down to phone width: corner panels never exceed 200px and never increase in width as the window narrows (R2); left and right clusters are equal width and symmetric about the center at every width (R1); the clusters stay pinned to the edges, with the center yielding symmetric gutters and no overlap, until the collision breakpoint (R3); at the breakpoint they reflow cleanly into the `#narrowStack` single column with no grow/center intermediate state (R4).
- CSS reflow and `initNarrowStack` reparent occur at the same viewport width (R5).
- Wide-desktop (≥1621px) and narrow single-column appearances match the previous build (R6).
- Verified in-browser (localhost `python3 -m http.server` + Claude-in-Chrome) at representative widths: 1620, 1400, 1180, 1080, ~breakpoint±1, 900, 480.
- Change applied to a new date-stamped build copy per the garage copy-first build workflow; README "Currently parked" line updated to match.

## Scope Boundaries

**Out of scope**
- The wide-desktop fixed layout appearance (≥1621px) and the narrow single-column stack *content/order* — preserved, not redesigned.
- Corner panel internal content (reaper rows, guild table, character chips) — untouched.
- Any change to `initNarrowStack`'s reparent/restore mechanism beyond the breakpoint constant.

### Deferred to Follow-Up Work
- Centralizing the shared breakpoint as a single build-time constant (currently duplicated between CSS media queries and the JS `matchMedia`, kept in sync by hand) — a nicety, not required for this fix.
