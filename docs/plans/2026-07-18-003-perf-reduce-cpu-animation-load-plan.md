---
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
product_contract_source: ce-brainstorm
execution: code
title: "perf: Reduce Chrome CPU Load (Animation Pass)"
type: perf
date: 2026-07-18
module: sooks-saga-scroll
origin: docs/plans/2026-07-18-003-perf-reduce-cpu-animation-load-plan.md
---

# perf: Reduce Chrome CPU Load (Animation Pass) — Plan

**Product Contract preservation:** unchanged — the Product Contract below is carried
verbatim from the requirements-only brainstorm; planning added only the Planning
Contract, Implementation Units, Verification Contract, and Definition of Done.

---

## Goal Capsule

- **Objective:** Cut the app's continuous Chrome CPU load by stopping the per-frame
  raster repaints that dominate it, while preserving the shimmer/glow visual identity
  where it reads as a reward.
- **Product authority:** User (owner of `sooks-saga-scroll`). Both aesthetic tradeoffs
  were resolved in brainstorm; the reaper-completion fork was resolved at plan time.
- **Open blockers:** None. Diagnosis is complete and evidence-backed (live browser
  profiling of build `sooks-saga-scroll-07182026-8.html`).

---

## Product Contract

### Context / Problem

Chrome runs hot when the app is open with a progressed character. Live profiling of
build `.8` isolated the cause:

- **Primary — 124 continuous `progressSheen` animations.** Each
  `.saga-progress-bar.has-progress .saga-progress-fill::after` animates
  `background-position` on a gradient every frame (~60fps) = one raster repaint per
  element per frame. `background-position` cannot be GPU-composited, so the cost lands
  on the main thread / raster. The twin-bars change (reaper + completion bar per saga)
  **doubled** this: a character progressed across all 62 sagas drives 124 simultaneous,
  forever-running sheens. Confirmed live: iteration `infinite`, `prefers-reduced-motion`
  false, running count 124.
- **Secondary (deferred, out of scope this pass).** All 517 quest rows render into the
  DOM even when 0 sagas are expanded (~15,200 nodes), and `container-type: inline-size`
  sits on every `.quest-block` (~579 containment contexts). This is a memory/reflow cost,
  not the continuous-CPU hog, and touches the load-bearing rollover — deferred.
- **Minor.** Box-shadow pulse keyframes (`lfmMidPulse`, `lfmHighPulse`, `lfmTagPulse`,
  `lfmRowPulse`, `ftrRowPulse`, `sagaNowPulse`) are the same repaint class but only fire
  on live-LFM / now-playing states.

### Goals

- Eliminate the continuous per-frame raster repaint load from progress-bar sheens.
- Preserve the shimmer as a **completion reward** — it still reads as "this bar is done."
- Keep any surviving animation GPU-cheap so it cannot re-introduce raster repaint cost.
- Keep the box-shadow pulse glows from being a repaint hog when their states are active.
- Prove the win: measurable before/after in-browser.

### Key Decisions

1. **Sheen gating (session-settled: user-directed — chosen over keep-all / GPU-only /
   static-only / one-shot-on-change).** The traveling shimmer runs **only on complete
   bars**. Partial/has-progress bars get a **static gold shine** (no animation). Shimmer
   becomes a completion-reward signal; the continuous animation count drops from 124 to
   single digits.
2. **GPU-cheap survivors (session-settled: user-directed).** The shimmer that remains on
   complete bars animates a compositor-friendly `transform` (`translateX` on the overlay)
   instead of `background-position`, so even the surviving handful composite on the GPU
   and force no raster repaints.
3. **Box-shadow pulses gated/cheapened (session-settled: user-directed).** The
   data-scaling pulse-glow keyframes are converted to a GPU-composited opacity pulse on a
   glow overlay so their active states no longer force per-frame raster repaints.
4. **Reaper "complete" = `sealed` (session-settled: user-approved at plan time — chosen
   over reaper-never-shimmers).** The reaper bar never reaches `.all-done`; its completion
   signal is `.sealed` (`ftrRemaining === 0`). A fully-reaped reaper bar is treated as
   complete and keeps the shimmer, mirroring the completion bar's `.all-done`.
5. **Secondary DOM/container work deferred (session-settled: user-directed — chosen over
   animation+lazy-render).** Lazy-rendering quest bodies and reducing the 579
   container-query contexts are out of scope; revisit only if CPU remains high after this
   pass is measured.

### Requirements

- **R1 — Partial bars are static.** A progress bar with progress but not complete shows a
  static gold shine and runs **no** continuous animation.
- **R2 — Complete bars keep the shimmer.** A fully-complete bar retains a visible
  traveling shimmer, visually equivalent to today's effect.
- **R3 — Surviving shimmer is GPU-composited.** The complete-bar shimmer animates a
  transform (not `background-position`), so it produces no per-frame raster repaint.
- **R4 — Both bar types covered.** The gating applies to **both** the reaper bar
  (complete = `.sealed`) and the completion bar (complete = `.all-done`).
- **R5 — Pulse glows bounded.** The **data-scaling** box-shadow pulse animations
  (`lfm-mid`, `lfm-high`, high LFM tag, high LFM quest row, first-time-reaper row) do not
  force continuous raster repaints; their active-state glow is delivered by a
  GPU-composited property (opacity/transform) rather than animated `box-shadow`. Singleton
  chrome pulses (`sagaNowPulse`, `addPulse`, `flowPulse`) are out of scope (see non-goals).
- **R6 — Respect reduced motion.** `prefers-reduced-motion: reduce` continues to suppress
  the surviving animations (existing behavior must not regress).
- **R7 — No visual regression on the polished work.** The header tier system, quest-row
  rollover, item/bar alignment, uniform pill widths, reaper text contrast, and LFM
  standout shipped in builds `.5`–`.8` render unchanged.

### Success Criteria / Acceptance Signals

- **Test fixture:** a character with progress in **every** saga (all ~124 bars
  `has-progress`) but only a **handful** of bars complete (`.all-done` / `.reaper.sealed`).
  This is the count-sensitive case — distinct from an all-complete character, where every
  bar stays animating (now as GPU transforms) and the count would not move.
- With that fixture, partial bars contribute **zero** running sheens; only the few complete
  bars animate — the continuous sheen count drops from **~124 to single digits**.
- No `background-position`-animated element remains among the continuously-running
  animations (verify via computed animation-name + property) — this is the primary
  raster-repaint win and holds regardless of the complete/partial mix.
- Complete bars still visibly shimmer; partial bars are static gold — confirmed by
  screenshot.
- Observable Chrome CPU reduction while the app sits open with the progressed character.
- `node --check` passes on the extracted script; builds `.5`–`.8` visual work intact.

### Scope Boundaries

**In scope:** sheen gating + GPU-cheap survivors, box-shadow pulse conversion,
before/after in-browser measurement.

#### Deferred to Follow-Up Work

- Lazy-rendering quest bodies (~15,200 DOM nodes) — separate optimization, only if CPU
  stays high after measuring this pass.
- Reducing the 579 container-query contexts on `.quest-block` — same follow-up; touches
  the load-bearing rollover.

#### Out of scope (non-goals)

- No storage-schema change (stays v14); no data/behavior change — presentation only.
- No new user-facing performance/reduce-motion toggle (`prefers-reduced-motion` is
  honored; a manual toggle is not added).
- Singleton chrome pulses (`sagaNowPulse`, `addPulse`, `flowPulse`) are not required to
  change — one instance each, negligible cost (may be converted opportunistically if
  trivial, but not a goal).

### Constraints

- Single ~16k-line offline HTML file; no build step, no module boundaries, no test
  framework.
- Garage copy-first + park-retention: edit a **copy** — next build
  `sooks-saga-scroll-07182026-9.html` — never the parked `.8`. Stamp the build id
  `07182026.9`. Retain 2 most recent + cross-day anchor; prune older to `_to_delete/`.
- Verification is `node --check` (node v26.5.0 installed) + in-browser via localhost
  server + Claude-in-Chrome (both gates work — see
  `docs/solutions/developer-experience/verify-single-file-html-without-node.md`).
- The `container-type` on `.quest-block` is load-bearing for the rollover; this pass must
  not touch it.

---

## Planning Contract

### Consolidated Research

Grounded against build `sooks-saga-scroll-07182026-8.html`:

- **Sheen definition** (lines ~1357–1376): `.saga-progress-bar.has-progress
  .saga-progress-fill::after` is a diagonal gold gradient sized `220% 100%`, offset to
  `220% 0`; animated by `@keyframes progressSheen { background-position: 220% 0 →
  -120% 0 }` at `3.2s linear infinite`, wrapped in `@media (prefers-reduced-motion:
  no-preference)`.
- **Completion state classes** (render, line 14369): completion bar gets `has-progress`
  when `st.done > 0` and `all-done` when `st.done >= st.total`.
- **Reaper state classes** (render, line 14362; `reaperSealed` at 14299): reaper bar gets
  `has-progress` when `st.reaperDone > 0` and `sealed` when `reaperTotal > 0 &&
  st.ftrRemaining === 0`. It **never** gets `all-done`. The reaper `has-progress` fill is
  a higher-specificity wax-red→gold override (lines 1333–1339).
- **Box-shadow pulses:** `lfmMidPulse`/`lfmHighPulse` on `.saga-progress-bar.lfm-mid`/
  `.lfm-high` (1404–1405), `lfmTagPulse` on `.lfm-skull-tag.high` (1490), `lfmRowPulse`
  on `.quest.lfm-high-row` (1510), `ftrRowPulse` on reaper rows (3611) — all animate
  `box-shadow`. Two guard patterns exist and must be preserved per rule: `lfmMid`/`lfmHigh`/
  `lfmTag`/`lfmRow` are wrapped in `@media (prefers-reduced-motion: no-preference)`;
  **`ftrRowPulse` is the opposite** — it animates unconditionally and is disabled by
  `@media (prefers-reduced-motion: reduce) { animation: none }` with a **distinct resting
  shadow** (`inset 0 0 10px rgba(212,171,90,0.55)`, ~line 3617) that must be kept.
  Also: `.saga-progress-bar` has `overflow: hidden` (line 1252), and the `lfm-mid`/
  `lfm-high` glow is a large **outer** `box-shadow` on the bar itself — relevant to U2.
  `sagaNowPulse` (1025), `addPulse` (402), `flowPulse` (789) are singleton chrome.
- **No test framework**; verification is syntax gate + browser visual/measurement pass.

### Key Technical Decisions

- **KTD1 — Gate the animation by completion class, keep `::after` on all has-progress
  bars.** The `::after` overlay stays present on every `has-progress` bar (it carries the
  shine); only the **animation** is scoped to the complete classes. Partial bars render
  the overlay as a fixed highlight (static shine). Implements Key Decision 1 + 4.
- **KTD2 — Animate `transform: translateX` on a fixed-width highlight band.** Replace the
  `220%`-wide background-position sweep with a narrower highlight band positioned by
  `transform`, animated `translateX` across the fill. Transform composites on the GPU →
  no raster repaint. Add `will-change: transform` only on the animated (complete) state to
  avoid creating idle compositor layers on every partial bar. Implements Key Decision 2.
  (session-settled KTD — see Key Decision 2.)
- **KTD3 — Deliver the pulse glow via an opacity-animated overlay.** For each data-scaling
  pulse selector, move the "hot" glow (the peak `box-shadow`) onto a pseudo-element
  overlay at full strength and animate its `opacity` between the trough and peak values.
  Opacity is GPU-composited; the static (max) box-shadow can remain as the resting look.
  Implements Key Decision 3. (session-settled KTD — see Key Decision 3.)
- **KTD4 — Selector for the static/animated split.** Animated shimmer targets
  `.saga-progress-bar.all-done .saga-progress-fill::after` and
  `.saga-progress-bar.reaper.sealed .saga-progress-fill::after`. Static shine targets
  `.saga-progress-bar.has-progress:not(.all-done):not(.sealed) .saga-progress-fill::after`.
  Exact selector grouping is the implementer's call as long as the behavior matches
  R1/R2/R4. (Deferred detail — see Open Questions.)

### High-Level Technical Design

Sheen decision, per bar, per state:

```
                          has-progress?      complete?              -> overlay behavior
completion bar   done>0 && done<total   NO (.all-done absent)   -> STATIC gold shine
completion bar   done>=total            YES (.all-done)         -> ANIMATED shimmer (transform)
reaper bar       reaperDone>0, !sealed  NO (.sealed absent)     -> STATIC gold shine
reaper bar       sealed (ftrRemaining=0) YES (.sealed)          -> ANIMATED shimmer (transform)
empty bar        no has-progress        n/a                     -> no overlay (quiet parchment)
```

Continuous-animation count: today every `has-progress` bar animates (124 across 62
sagas). After: only `.all-done` + `.reaper.sealed` bars animate, and those via transform
(GPU), so the raster-repaint count for progress sheens drops to **zero**.

---

## Implementation Units

### U1. Gate the progress-bar sheen and make survivors GPU-cheap

- **Goal:** Traveling shimmer runs only on complete bars (`.all-done` and
  `.reaper.sealed`); partial `has-progress` bars show a static gold shine; the surviving
  shimmer animates a `transform` instead of `background-position`.
- **Requirements:** R1, R2, R3, R4, R6; R7 (regression guard — verified in Verification
  Contract step 6). Cites KTD1, KTD2, KTD4 and Key Decisions 1, 2, 4.
- **Dependencies:** none (first content change after the build copy is made).
- **Files:** `sooks-saga-scroll-07182026-9.html` (copy of `.8`; CSS block ~lines
  1357–1385, `@keyframes progressSheen`).
- **Approach:**
  - Keep the `::after` overlay on all `has-progress` bars but split its treatment:
    - **Static (partial):** a fixed soft-gold highlight (a centered/low-offset gradient or
      a resting transform position) with **no animation** — the "static gold shine."
    - **Animated (complete):** rebuild the overlay as a fixed-width highlight band and
      animate `transform: translateX(...)` across the fill; replace the `progressSheen`
      background-position keyframes with a transform keyframe. Apply the animation only to
      `.all-done` and `.reaper.sealed`, inside the existing `@media (prefers-reduced-motion:
      no-preference)` guard. Add `will-change: transform` on the animated state only.
  - The reaper `has-progress` fill override (wax-red→gold, lines 1333–1339) is untouched;
    only the `::after` animation gating changes.
- **Execution note:** No unit-test framework — verify by browser measurement + screenshot
  (see Verification Contract). Prefer confirming the running-animation count drop and the
  static-vs-animated visual split directly in Chrome before parking.
- **Patterns to follow:** existing `@media (prefers-reduced-motion: no-preference)` guard
  pattern already used for the sheen and the pulse animations; existing higher-specificity
  reaper override pattern (lines 1333–1339) for keeping reaper fill distinct.
- **Test scenarios:**
  - Covers R2/R4. A completion bar at `done >= total` (`.all-done`) shows a moving
    shimmer; a reaper bar at `ftrRemaining === 0` (`.reaper.sealed`) shows a moving
    shimmer.
  - Covers R1/R4. A completion bar with `0 < done < total` shows a static gold shine and
    **no** running animation; a reaper bar with progress but not sealed shows the same.
  - Covers R3. On a complete bar, the running animation's animated property is `transform`
    (not `background-position`) — confirm via `getComputedStyle`/animation inspection.
  - Covers R6. Under `prefers-reduced-motion: reduce`, no shimmer animates; complete bars
    fall back to the static highlight.
  - Empty (no `has-progress`) bars render no overlay and no animation (unchanged).
- **Verification:** With a fully-progressed test character, the count of running sheen
  animations is single digits (only complete bars), none animate `background-position`,
  partial bars are visibly static gold, complete bars visibly shimmer, and `.5`–`.8`
  visual work is intact.

### U2. Convert data-scaling box-shadow pulses to GPU-composited opacity pulses

- **Goal:** The pulse glows that scale with live data (`lfm-mid`, `lfm-high`, high LFM
  tag, high LFM quest row, first-time-reaper row) no longer animate `box-shadow`; their
  pulsing glow is delivered by an opacity-animated overlay so active states force no
  per-frame raster repaint.
- **Requirements:** R5, R6. Cites KTD3 and Key Decision 3.
- **Dependencies:** none functionally; sequence after U1 to keep one CSS change per commit
  and simplify before/after measurement.
- **Files:** `sooks-saga-scroll-07182026-9.html` (CSS: `lfmMidPulse`/`lfmHighPulse`
  ~1393–1409, `lfmTagPulse` ~1490, `lfmRowPulse` ~1510, `ftrRowPulse` ~3611 and their
  selectors).
- **Approach:** For each selector, set the resting look to the current max/hot
  `box-shadow` (static), add an overlay carrying the peak glow, and animate its `opacity`
  between trough and peak in place of the `box-shadow` keyframes — keeping the same
  durations/easing and each rule's **existing** reduced-motion behavior (see below).
  Singleton chrome pulses (`sagaNowPulse`, `addPulse`, `flowPulse`) are out of scope and
  left as-is unless the same overlay swap is trivially reusable.
  - **Outer-glow clipping (the two bar pulses).** `.saga-progress-bar` has
    `overflow: hidden` (line 1252), and `lfm-mid`/`lfm-high` render a large **outer** glow.
    A child pseudo-element's outer `box-shadow` would be clipped to the bar rectangle,
    collapsing the halo. Anchor the opacity-animated glow overlay for these two on a
    **non-clipping ancestor** (e.g. `.saga-barwrap`), not on a pseudo-element inside the
    bar. The row/tag pulses (`.lfm-skull-tag.high`, `.quest.lfm-high-row`, reaper rows)
    have no `overflow: hidden` and can use an in-element pseudo-element.
  - **Reduced-motion parity per rule.** `lfmMid`/`lfmHigh`/`lfmTag`/`lfmRow` use the
    `@media (prefers-reduced-motion: no-preference)` guard — keep it. **`ftrRowPulse` is
    the opposite**: it animates unconditionally and is disabled by `@media
    (prefers-reduced-motion: reduce)` with a distinct resting shadow
    (`inset 0 0 10px rgba(212,171,90,0.55)`); preserve that exact reduced-motion resting
    value when converting it (R6).
- **Execution note:** These states are live-data-driven (reaper-tier LFM / now-playing);
  reproduce them in-browser by monkeypatching the LFM supplier (`window.lfmForTier`, per
  the verification doc) so the pulse selectors actually apply, then confirm the pulse
  still reads and its animated property is `opacity`.
- **Patterns to follow:** the `::after` overlay pattern used for the sheen in U1; existing
  reduced-motion guards around each pulse.
- **Test scenarios:**
  - Covers R5. With a `lfm-high` container bar present, the pulse still visibly cycles and
    its running animation animates `opacity` (not `box-shadow`).
  - Covers R5. Same for a high LFM tag (`.lfm-skull-tag.high`), a high LFM quest row
    (`.quest.lfm-high-row`), and a first-time-reaper row (`ftrRowPulse`).
  - Covers R6. Under `prefers-reduced-motion: reduce`, none of these pulses animate; the
    resting (max) glow remains.
  - When no live-LFM / reaper-row state is present, none of these animations run (no
    regression to the idle case).
- **Verification:** With the LFM states injected, the pulse glows read as before but every
  running pulse animates `opacity`; with them absent, no pulse animations run; reduced
  motion suppresses all of them.

---

## Verification Contract

No automated test framework exists; verification is a syntax gate plus an in-browser
measurement/visual pass (per
`docs/solutions/developer-experience/verify-single-file-html-without-node.md`).

1. **Syntax gate.** Extract the inline script and run `node --check`:
   `awk '/<script>/{f=1;next} /<\/script>/{f=0} f' sooks-saga-scroll-07182026-9.html >
   /tmp/app.js && node --check /tmp/app.js` → `SYNTAX_OK`.
2. **Baseline measurement (before, on `.8`).** Serve over localhost, load in Chrome,
   inject the test-fixture character (progress in every saga, few bars complete — see
   Success Criteria), and record the running sheen-animation count (expect ~124) as the
   baseline reference.
3. **After measurement (on `.9`).** Repeat with the same fixture on the new build:
   running sheen-animation count is **single digits**, and none of the
   continuously-running progress-sheen animations animate `background-position` (they
   animate `transform`).
4. **Visual split.** Screenshot confirms complete bars (`.all-done`, `.reaper.sealed`)
   shimmer; partial `has-progress` bars are static gold; empty bars unchanged.
5. **Pulse check.** With LFM states injected, pulse glows still read and animate `opacity`;
   with them absent, no pulses run.
6. **Regression pass.** Header tiers, quest-row rollover, bar/pill alignment, reaper text
   contrast, and LFM standout from builds `.5`–`.8` render unchanged (screenshot at a
   narrow width to exercise the rollover).
7. **Reduced-motion pass.** With `prefers-reduced-motion: reduce`, no shimmer or pulse
   animates; static fallbacks show.
8. **Cleanup.** Remove injected styles/character, `localStorage.removeItem('sooksSagaScroll')`,
   stop the server.

---

## Definition of Done

- U1 and U2 implemented in `sooks-saga-scroll-07182026-9.html` (copy-first from `.8`;
  build id stamped `07182026.9`).
- Verification Contract steps 1–7 pass; before/after animation counts recorded (124 →
  single digits) and the CPU reduction observed.
- No `background-position`-animated element and no `box-shadow`-animated data-scaling pulse
  remains among continuously-running animations.
- Builds `.5`–`.8` visual work verified intact; no schema change (still v14).
- Park per garage retention (live `.9` + `.8` + cross-day anchor `07142026-2`; prune older
  to `_to_delete/`); update README park-state line, build ladder, and Resume prompt.

---

## Open Questions (deferred to implementation)

- **Exact static-shine treatment for partial bars.** Whether the static gold shine is a
  centered fixed gradient, a resting transform position of the highlight band, or a
  simplified static overlay — implementer's call as long as it reads as a gold shine and
  runs no animation (R1). (KTD4.)
- **Overlay reuse for pulses.** Whether each pulse selector gets its own overlay element
  or a shared glow-overlay convention — an implementation-time simplification decision;
  behavior (R5) is the contract, not the element count.
- **`will-change` scope.** Confirm in-browser that applying `will-change: transform` only
  to the complete/animated state (not all has-progress bars) avoids creating idle
  compositor layers; adjust if layer count regresses.
