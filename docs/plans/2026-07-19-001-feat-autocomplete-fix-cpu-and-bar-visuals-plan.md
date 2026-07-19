---
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
execution: code
product_contract_source: ce-plan-bootstrap
title: "feat: Autocomplete reaper fix, DS&A CPU pass, and progress-bar visuals"
date: 2026-07-19
plan_type: feat
---

# feat: Autocomplete Reaper Fix, DS&A CPU Pass, and Progress-Bar Visuals — Plan

**Target file:** `sooks-saga-scroll-07182026-9.html` (single-file app; the build
will be copied forward to a new date/iteration filename before editing, per the
garage copy-first workflow). All line numbers below reference the `.9` build.

## Summary

A five-item batch on Sook's Saga Scroll: one autocomplete correctness bug, a
JavaScript data-structure/algorithm pass to lower render CPU, and three
progress-bar visual changes. No schema change (stays v14). CSS + JS only.

The DS&A changes (U2–U3) produce **byte-identical rendered output** — they only
remove redundant per-render work. The three visual changes (U4–U6) alter
appearance and require the user's in-browser pass (browser automation is blocked
on this project; Claude verifies syntax only).

---

## Problem Frame

1. **Reaper autocomplete re-celebrates a claimed First-Time Reaper.** In
   `_acApply`, an already-reapered quest whose green completion seal was cleared
   (saga redeemed → new cycle) re-enters the `wantReaper` path on a re-run,
   re-adds to `reaperPopQueue`, and toasts "First-Time Reaper sealed
   automatically" — but FTR is a one-time claim, already banked. It should mark
   the quest complete plainly.
2. **Render hot path does O(sagas) / O(sagas×quests) work per row.**
   `renderSagaList()` tears down and rebuilds all ~53 cards / ~479 quest rows on
   every click, autocomplete, and poll tick. Per-row linear scans
   (`SAGAS.find`, `lookupQuestDetails` normalized fallback, double
   `normalizeQuestName`/`lfmForTier`) multiply across ~479 rows.
3. **LFM signal is doubled on the saga header.** The header's `.lfm-skull-tag`
   indicator glows AND the container progress bar glows (`.lfm-mid`/`.lfm-high`).
   The bar glow is redundant.
4. **Reaper fill's leading edge washes out.** The reaper fill ends light
   (`--gold-light` / `--wax-red`), so as it advances L→R the right edge blends
   into the light unfilled track — the progress boundary reads soft.
5. **Partial bars shimmer.** Every `has-progress` bar shows a static gold shine
   band; the user wants partial bars solid (flat, no gradient, no shine) and the
   shimmer reserved for completed bars only.

---

## Requirements

- **R1** — Reaper autocomplete on an already-reapered quest (FTR banked, seal
  cleared) marks the quest complete (green seal, cascade, auto-bank) and toasts
  a plain completion; it does **not** re-fire the ☠ pop or the reaper toast.
- **R2** — All autocomplete regression paths (fresh reaper seal, fully-done
  no-op, normal mode, NO_REAPER fallback) are unchanged.
- **R3** — Render-path CPU is reduced by removing redundant per-render linear
  scans, with **byte-identical** rendered DOM output.
- **R4** — The saga-header container progress bar no longer glows for LFM; the
  `.lfm-skull-tag` indicator glow is retained.
- **R5** — The reaper fill has a crisp, darker leading edge that separates it
  from the unfilled track at any progress level.
- **R6** — Partial (has-progress, not complete) bars render a flat solid fill
  with no shine band; only complete bars (`.all-done` / `.reaper.sealed`) keep
  the gradient + animated shimmer.

---

## Key Technical Decisions

- **KTD1 — Fix the bug by gating `wantReaper` on `!alreadyReaper`, not by
  special-casing the guard.** *(session-settled: user-directed — "it should just
  mark the quest as complete".)* Reorder so `alreadyReaper` is computed before
  `wantReaper`, then `wantReaper = mode === "reaper" && !NO_REAPER.has(qName) &&
  !alreadyReaper`. This makes the whole function treat an already-reapered quest
  as a normal completion: green seal cascades, `reaperPopQueue.add` is skipped,
  and the return object's `reaper` is false so the caller shows the plain toast.
  The existing guard `if (alreadyDone && (!wantReaper || alreadyReaper)) return
  null;` still short-circuits the fully-done case correctly (the `|| alreadyReaper`
  clause becomes redundant but harmless).
- **KTD2 — Memoize only static, load-time data; no runtime invalidation.**
  *(session-settled: user-directed — CPU pass, byte-identical output.)* `SAGAS`,
  `QUEST_DETAILS`, `SHARED_QUESTS` are constant after load, so `SAGA_BY_ID`, the
  `QUEST_DETAILS` exact/normalized indexes, `saga._normQuests`, and the
  `normalizedName → saga[]` index are built once and never invalidated. This
  keeps the change provably output-preserving.
- **KTD3 — Structural single-card re-render (#4) is out of scope.**
  *(session-settled: user-directed — "safe batch, defer #4".)* Re-rendering only
  the affected saga card on a toggle is the largest CPU win but changes the
  render/event-wiring flow and carries regression surface; deferred to its own
  build (see Scope Boundaries).
- **KTD4 — Kill the LFM bar glow at the class-application site, not just the
  CSS.** *(session-settled: user-directed.)* Drop `${lfmSkullClass}` from the
  container bar's class list (line 14441) so `.lfm-mid`/`.lfm-high` never attach
  to the bar; the `.lfm-skull-tag` indicator computes its own class
  independently and is untouched. The now-dead `.saga-progress-bar.lfm-mid/high`
  CSS (1404–1436) is removed to keep the sheet honest.
- **KTD5 — Partial = flat solid; complete = gradient + shimmer.**
  *(session-settled: user-directed — "partial bars should be flat".)* Partial
  completion fill → solid green; partial reaper fill → solid wax-red; the
  `::after` shine band renders only for `.all-done` / `.reaper.sealed`.

---

## Implementation Units

### U1. Fix reaper autocomplete re-fire on already-reapered quests

- **Goal:** An already-reapered quest with a cleared completion seal marks
  complete plainly on a reaper-mode re-run — no false ☠ pop or reaper toast.
- **Requirements:** R1, R2 (cites KTD1)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — `_acApply` (~13060–13089)
- **Approach:** Compute `alreadyReaper` before `wantReaper`; set `wantReaper =
  mode === "reaper" && !NO_REAPER.has(qName) && !alreadyReaper`. Leave the rest
  of the cascade untouched — because `wantReaper` is now false for an
  already-reapered quest, the FTR-seal write and `reaperPopQueue.add(qName)` are
  skipped and the returned `reaper` is false, so `_autoCompleteCheck` (~13135)
  emits the "marked Quest Completed automatically" toast. Optionally simplify the
  guard to `if (alreadyDone && !wantReaper) return null;` (behavior-equivalent).
- **Patterns to follow:** existing `_acApply` cascade; `applyQuestState` /
  `setSagaDoneFor` semantics it mirrors.
- **Test scenarios** (manual, in-browser, driven by the user's visual pass):
  - Quest never completed, mode reaper, not NO_REAPER → seals green + reaper,
    ☠ pop fires, toast reads "First-Time Reaper sealed". (regression — unchanged)
  - Quest with `quests[q]="reaper"` AND `sagaDone[q]=true`, mode reaper → no-op,
    no toast. (regression — unchanged)
  - Quest with `quests[q]="reaper"` but `sagaDone[q]` cleared, mode reaper →
    green seal set + cascade + auto-bank; `quests[q]` stays "reaper"; **no** ☠
    pop; toast reads "marked Quest Completed automatically". (the fix)
  - NO_REAPER quest, mode reaper → normal completion only. (regression)
  - Mode normal, quest already reaper, not done → green seal only, reaper
    untouched. (regression)
- **Verification:** `jsc`/`node --check` syntax OK; user confirms the four
  behaviors above in a browser with a crafted test character.

### U2. Index static data for O(1) lookups (SAGA_BY_ID, QUEST_DETAILS, normalized-name → saga[])

- **Goal:** Remove the per-row linear scans over `SAGAS` and `QUEST_DETAILS` and
  the O(sagas×quests) `_sagaForQuest` scan, with identical output.
- **Requirements:** R3 (cites KTD2)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — JS module scope (near `SAGAS` /
  `QUEST_DETAILS` definitions); `lookupQuestDetails` (~15226); `_sagaForQuest`
  (~16252); `sagasMatchingArea` (~13919); call sites 13081, 14228, 14548, 14580,
  15019, 15078, 15322, 15392.
- **Approach:**
  - Build `const SAGA_BY_ID = new Map(SAGAS.map(s => [s.id, s]))` once at load;
    replace every `SAGAS.find(s => s.id === …)` with `SAGA_BY_ID.get(…)`.
  - Precompute two `QUEST_DETAILS` indexes at load: an exact-name map and a
    normalized-name map (`normalizeQuestName(key) → details`). Rewrite
    `lookupQuestDetails`'s case-2/case-3 fallbacks to two O(1) lookups.
  - Build a `normalizedQuestName → saga[]` index at load; use it in
    `_sagaForQuest` and `sagasMatchingArea` in place of the
    `SAGAS.filter(... .some(normalizeQuestName …))` scans.
- **Patterns to follow:** existing top-level `const` table definitions; keep the
  indexes adjacent to their source tables with a short comment.
- **Test scenarios:** Test expectation: behavior-preserving refactor — verify
  **identical** output, not new behavior. Spot-check: a quest with an exact
  detail entry, a quest resolved only via normalized fallback, a shared-quest
  sibling, and a reaper-corner LFM row all render exactly as before. Confirm the
  15s poll and a checkbox toggle still mark/cascade identically.
- **Verification:** `jsc` syntax OK; user spot-checks a few sagas + one reaper
  LFM row look unchanged; no console errors.

### U3. Trim redundant per-render work (normalized names, lfmForTier, sagaStatus, LFM signature)

- **Goal:** Cut the duplicate `normalizeQuestName`/`lfmForTier` passes and the
  triple `sagaStatus` filter, and stop re-serializing the unchanged LFM index
  each tick — identical output.
- **Requirements:** R3 (cites KTD2)
- **Dependencies:** U2 (shares the normalized-name indexing)
- **Files:** `sooks-saga-scroll-*.html` — `renderSagaCard` (~14391 tint loop),
  `renderQuestRow` (~15048), `lfmForTier` (~13759), `sagaStatus` (~12655),
  `refreshLfmIndex` (~13681).
- **Approach:**
  - Precompute `saga._normQuests` (array of normalized quest names) once at load
    so renders never re-regex static names.
  - Compute `lfmForTier` once per quest in `renderSagaCard` and thread the result
    into `renderQuestRow` instead of recomputing there.
  - Fold `sagaStatus`'s three `.filter` passes into a single loop accumulating
    `done`, `reaperDone`, `reaperEligibleTotal`.
  - Cache the last `_lfmIndex` signature string; compare the new signature
    against the cached value instead of rebuilding the old one every tick.
- **Patterns to follow:** existing render helpers; keep threaded values as
  explicit function params.
- **Test scenarios:** Test expectation: behavior-preserving — verify identical
  output. Spot-check a saga with a live mid-tier LFM and one with a high-tier
  LFM render the same header tint and row highlight as before; confirm the LFM
  index still updates when an LFM appears/disappears across a poll.
- **Verification:** `jsc` syntax OK; user confirms LFM tinting + row highlights
  unchanged across a poll; no console errors.

### U4. Remove the LFM glow from the container progress bar

- **Goal:** The saga-header bar no longer glows for LFM; the `.lfm-skull-tag`
  indicator keeps its glow.
- **Requirements:** R4 (cites KTD4)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — bar class assembly at line 14441; CSS
  `.saga-progress-bar.lfm-mid` / `.lfm-high` and their `::before` (1404–1436).
- **Approach:** Drop `${lfmSkullClass}` from the container bar's className at
  14441 so the LFM classes never attach to the bar. Remove the now-dead
  `.saga-progress-bar.lfm-mid`/`.lfm-high` rules and their `::before` pulse
  overlay. Leave the `.lfm-skull-tag` indicator styling and its own class
  computation (14401) untouched.
- **Patterns to follow:** the `.lfm-skull-tag` indicator remains the single LFM
  glow on the header.
- **Test scenarios:** With a saga showing a live mid- and high-tier LFM: the
  skull-tag indicator glows; the progress bar shows **no** orange/red glow,
  border, wash, or pulse. Non-LFM sagas unchanged.
- **Verification:** `jsc` syntax OK; user confirms indicator-only glow in a
  browser.

### U5. Sharpen the reaper fill's leading edge

- **Goal:** The reaper fill's right (leading) edge reads crisply against the
  unfilled track at any progress level.
- **Requirements:** R5 (cites KTD1's sibling visual decision)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — CSS `.saga-progress-bar.reaper
  .saga-progress-fill` (1304) and `.reaper.has-progress .saga-progress-fill`
  (1333).
- **Approach (directional):** Give the reaper fill a defined dark leading edge —
  e.g. an inset right-edge shadow (`inset -Npx 0 … rgba(dark-wax-red)`) or a thin
  `border-right`/hard gradient stop in a dark wax-red — so the fill/track boundary
  is a sharp rule rather than a light wash. Implementer picks the cleanest of
  these that survives the bar's `overflow: hidden` and the rounded corners.
  Applies to the partial reaper fill (post-U6 solid) as well as any progress
  level; keep the cream count text + halo (1313) legible over the darker edge.
- **Test scenarios:** Reaper bar at ~25%, ~50%, ~90% progress — the leading edge
  is visibly darker/crisper than the track at each; count text stays legible;
  fully-sealed reaper bar unaffected.
- **Verification:** `jsc` syntax OK; user confirms edge contrast at multiple
  progress levels.

### U6. Flat solid partial bars; shimmer only when complete

- **Goal:** Partial bars are solid (flat color, no gradient, no shine); complete
  bars keep the gradient + animated shimmer.
- **Requirements:** R6 (cites KTD5)
- **Dependencies:** U5 (both touch reaper fill CSS — land U5 then U6, or
  co-verify)
- **Files:** `sooks-saga-scroll-*.html` — CSS `.has-progress .saga-progress-fill`
  (1349), `.reaper.has-progress .saga-progress-fill` (1333),
  `.has-progress .saga-progress-fill::after` (1366), `.all-done` (1391) /
  `.reaper.sealed` fill rules.
- **Approach:**
  - Partial completion fill (`has-progress`, not `all-done`) → a flat solid green
    (drop the green→gold gradient).
  - Partial reaper fill (`reaper.has-progress`, not `sealed`) → a flat solid
    wax-red (drop the wax-red→gold gradient), preserving U5's leading edge.
  - Scope the `::after` shine band to `.all-done` / `.reaper.sealed` only, so
    partial bars have no shine.
  - `.all-done` and `.reaper.sealed` keep their existing gradient fills and the
    complete-only `progressSheen` animation (1377–1387) — unchanged.
- **Patterns to follow:** existing complete-bar gradient/shimmer rules stay as
  the "completed" treatment; partial becomes the quiet solid state.
- **Test scenarios:** Partial completion bar → solid green, no shine, no
  shimmer. Partial reaper bar → solid wax-red with the U5 leading edge, no shine.
  Fully-complete bar (`all-done`) → gradient + moving shimmer. Fully-sealed
  reaper bar → gradient + shimmer. `prefers-reduced-motion` → complete bars fall
  back to the static shine (unchanged).
- **Verification:** `jsc` syntax OK; user confirms the partial-vs-complete split
  in a browser and checks reduced-motion.

---

## Scope Boundaries

**In scope:** U1–U6 above.

### Deferred to Follow-Up Work

- **Structural single-card re-render (DS&A finding #4).** Re-render only the
  affected saga card(s) on a quest toggle instead of rebuilding the whole
  ~479-row list. Largest CPU win, but it changes the render + `wireCard`
  event-wiring flow and needs a careful full-interaction visual pass — its own
  build. (KTD3)

---

## Verification Contract

- **Syntax:** `jsc` (or `node --check` if available) reports `SYNTAX_OK` on the
  new build. Node is not installed on this machine; `jsc` is the syntax gate.
- **DS&A units (U2, U3):** output-preserving — the bar is met by confirming
  identical rendering + identical mark/cascade behavior, not new behavior.
- **Visual units (U1 behaviors, U4, U5, U6):** browser automation is blocked on
  this project, so Claude cannot self-verify visually — the in-browser pass is
  the **user's** per the project's standing workflow. Claude verifies syntax and
  hands off a crafted test-character recipe for U1.
- **No schema change:** state stays v14; `sooksSagaScroll` storage untouched.

## Definition of Done

- All six units implemented in a copied-forward build file (copy-first, never
  edit the parked `.9` in place).
- Build version stamped (mmddyyyy.x) per the build-versioning skill.
- `jsc` syntax passes.
- README "Currently parked" line + build note updated; park retention applied
  (2 most recent overall + cross-day anchor) per garage-park-retention.
- User has done the in-browser visual pass on U1/U4/U5/U6 (deferred to them).
