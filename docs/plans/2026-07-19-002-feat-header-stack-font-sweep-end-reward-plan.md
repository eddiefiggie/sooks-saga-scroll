---
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
execution: code
product_contract_source: ce-brainstorm
title: "feat: Saga header/LFM layout, font sweep, filters redesign, zebra, and completed progress bar"
date: 2026-07-19
plan_type: feat
---

# feat: Saga Header/LFM Layout, Font Sweep, Filters Redesign, Zebra, and Completed Progress Bar — Plan

**Target file:** `sooks-saga-scroll-07192026-1.html` (single-file app; copy forward
to a new build filename before editing, per the garage copy-first workflow). Line
numbers reference the `-1` build. CSS + JS-markup only; no schema change (v14).

**All designs in this plan were prototyped and validated in-browser during planning**
(localhost + Claude-in-Chrome, on a crafted test character). The CSS/markup approaches
below are the ones proven to render correctly; the build's visual pass re-confirms them.

## Summary

Seven user-directed UI changes, each approved from a live in-browser draft:
(U1) restructure the saga/non-saga header so the level pill + title get their own row,
the reward pill + progress bars drop to a second row, and the LFM notification fills
the reclaimed right-side space (snapping to a full-width bottom row when narrow);
(U2) retire the hard-to-read IM Fell fonts from all reading text → upright EB Garamond;
(U3) remove the redundant in-body "End Reward" control; (U4) wrap quest-row pills
cleanly when narrow instead of overflowing; (U5) add pronounced themed zebra striping
to quest rows (active and completed stash); (U6) redesign the filters container into
bounded, uniform subcontainer panels with the fold intact and Expand All removed;
(U7) render the collapsed COMPLETED rollup as a fractional green→gold completion
progress bar with bold black text.

---

## Problem Frame

1. **Header crowding.** `.saga-header` (line 1138) is a 2-col grid (`"t1 t2"`) — the
   level+title tier and the reward-pill+bars tier share one row, truncating nearly
   every title. The reclaimed right-side space (once stacked) is unused, and the LFM
   bar sits on its own full row.
2. **Unreadable IM Fell.** `.saga-story-body` (~2901) = `IM Fell DW Pica` italic;
   `--font-flavor` (~35) = `IM Fell English` drives ~9 spots including
   `.completed-rollup-count` (italic, 11.5px, faded — the worst offender).
3. **Redundant reward control.** The opened body renders `.reward-controls` ("End
   Reward" label + `.reward-btn` pill, render ~14549), duplicating the header pill.
4. **Pill overflow.** On narrow widths the quest-row pills in `.completion-controls`
   spill outside the card.
5. **No quest separation.** Quest rows blend together with no row banding.
6. **Loose filters.** Filter section labels float over their options with no clear
   boundaries; an unwanted "Expand All" button sits among them.
7. **Faint completed rollup.** The collapsed "COMPLETED — N quests tucked away" bar
   barely registers against the parchment.

---

## Requirements

- **R1** — Header row 1 = level pill + title only; row 2 = reward pill + both bars.
- **R2** — At wide width the LFM notification fills the right-side space spanning both
  header rows; when narrow, it snaps to a full-width bottom row. Sagas without an LFM
  leave no reserved right-side gap.
- **R3** — The reward pill bottom-aligns with the progress bars (not the captions).
- **R4** — No reading text renders in IM Fell; the tale-so-far and completed count are
  upright, clearly legible EB Garamond. No new font introduced.
- **R5** — The in-body "End Reward" control (label + pill) is gone; the header reward
  pill still toggles banked/incomplete.
- **R6** — Quest-row pills wrap cleanly inside the card on narrow widths.
- **R7** — Quest rows carry pronounced, themed alternating banding — in the active list
  and in the expanded completed stash.
- **R8** — Filter sections are bounded, uniform-sized subcontainer panels (Group By,
  Search, Audio Alerts, Quest Autocomplete same size, options wrapping within); the
  Tier panel is full-width; the fold still collapses the whole container; Expand All is
  removed with its supporting code cleaned up.
- **R9** — The collapsed COMPLETED rollup reads as a completion progress bar: green→gold
  fill proportional to completed ÷ total quests in the saga (like the header Complete
  bar), empty track for the remainder, crisp leading edge, bold black legible text.

---

## Key Technical Decisions

- **KTD1 — Header becomes a 2-column grid with the LFM spanning both rows on the right.**
  *(session-settled: user-directed.)* `grid-template-areas: "t1 t3" "t2 t3"` with a
  flexible left column and an `auto` right column; `.hdr-t3` overrides its current
  `grid-column: 1 / -1` to take the `t3` area and `align-self: stretch`. When no LFM
  element renders, the `auto` column collapses and the left column takes full width.
  The narrow `@container` breakpoint restacks to `"t1" "t2" "t3"` with `.hdr-t3`
  full-width again. Reuses the existing narrow-stack rules as the target shape.
- **KTD2 — Fix pill/bar alignment with `align-items: flex-end` on `.hdr-t2`.**
  *(session-settled: user-directed.)* The pill must bottom-align with the bars, not
  center against the taller caption+bar column (centering was the visible bug).
- **KTD3 — Sweep IM Fell by repointing `--font-flavor` AND forcing upright on the two
  worst spots.** *(session-settled: user-directed — "sweep app-wide".)* Repoint
  `--font-flavor` to the EB Garamond stack (sweeps ~9 consumers); additionally set
  `.saga-story-body` and `.completed-rollup-count` to the EB Garamond stack with
  `font-style: normal` — the variable repoint alone left them italic/tiny/faded and
  still hard to read (verified: `.completed-rollup-count` computed to IM Fell italic
  until forced upright).
- **KTD4 — Remove the in-body End Reward control (markup + handler + CSS); keep the
  header pill.** *(session-settled: user-directed.)*
- **KTD5 — Quest zebra rides `.quest-block:nth-*` so it covers active and completed-stash
  quests uniformly.** *(session-settled: user-directed — "quest separation is perfect".)*
  The completed quests are `.quest-block` inside `.completed-rollup-body`, so the same
  rule bands them.
- **KTD6 — Filters restructure keeps everything inside the foldable `.ctl-row` region.**
  *(session-settled: user-directed.)* The fold hides `.ctl-row` children via
  `.controls.folded`; the 4 small panels must live in a grid that folds with the
  container (a preview `display:grid !important` outside that mechanism broke the fold —
  the real markup must keep the grid within a `.ctl-row` or extend the fold rule). The
  four small panels (Group By, Search, Audio, Autocomplete) are a uniform 2×2 grid with
  equal min-height and options wrapping inside; Tier is a full-width panel; **Expand All
  is removed entirely** (button, handler, and `state.expandAll` wiring), not relocated.
- **KTD7 — Collapsed COMPLETED bar is a fractional progress bar.**
  *(session-settled: user-directed — "based on the fraction of total completed quests
  versus quests available in the saga … just like the Complete pill in the saga
  header".)* Fill % = `sagaStatus().done / total`, computed at render time like the
  header bar; green→gold fill + empty track + crisp leading edge + gold trim + bold
  black text.

---

## Implementation Units

### U1. Restructure the saga/non-saga header (title row / actions row / LFM right-panel)

- **Goal:** Level pill + title on row 1; reward pill + bars on row 2; LFM filling the
  right at wide width, snapping to a full-width bottom row when narrow. No title
  truncation; pill bottom-aligned with the bars.
- **Requirements:** R1, R2, R3 (cites KTD1, KTD2)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — CSS `.saga-header` (~1138), `.hdr-t1`/`.hdr-t2`
  (~1159-1163), `.hdr-t3` + `.hdr-t3 .lfm-skull-tag` (~1170), the `@container
  (max-width: …)` block (~1200).
- **Approach:** Base grid → `grid-template-columns: minmax(0,1fr) auto` with
  `grid-template-areas: "t1 t3" "t2 t3"`. `.hdr-t2` → `justify-self: start;
  align-items: flex-end` (KTD2 alignment fix). `.hdr-t3` → assign `grid-area: t3`,
  override `grid-column` off `1 / -1`, `align-self: stretch; justify-self: end`, and
  restyle `.hdr-t3 .lfm-skull-tag` from a full-width bar into a right-side panel
  (`width: auto; max-width ~280px; height: 100%`, centered content). Narrow
  `@container` → `grid-template-areas: "t1" "t2" "t3"`, `.hdr-t3` back to
  `grid-column: 1/-1; justify-self: stretch` and the tag back to `width: 100%`. Tune the
  breakpoint during the visual pass (the side-panel should collapse before it crowds the
  bars). Shared `.saga-header` covers saga and non-saga containers.
- **Patterns to follow:** the existing `@container (max-width:470px)` stacked rules are
  the narrow-state reference; the header LFM full-width bar (build 07182026.6) is the
  narrow-row target.
- **Test scenarios** (in-browser): long-title saga renders full title on row 1, pill+bars
  on row 2; a saga WITH an LFM shows the right-side panel spanning both rows; a saga
  WITHOUT an LFM leaves no right gap; non-saga container stacks identically; narrow width
  snaps the LFM to a full-width 3rd row; pill sits level with the bars.
- **Verification:** `node --check` SYNTAX_OK; in-browser at wide + narrow on saga + non-saga.

### U2. Sweep IM Fell → upright EB Garamond across all reading text

- **Goal:** No reading text in IM Fell; tale-so-far and completed count legible.
- **Requirements:** R4 (cites KTD3)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — CSS `--font-flavor` (~35), `.saga-story-body`
  (~2901), `.completed-rollup-count` (~2117).
- **Approach:** Repoint `--font-flavor` to `'EB Garamond','IM Fell English',Georgia,serif`
  (mirrors `--font-body`), sweeping all `var(--font-flavor)` consumers. Additionally set
  `.saga-story-body` and `.completed-rollup-count` to the EB Garamond stack with
  `font-style: normal` (KTD3 — the variable repoint alone left them italic and hard to
  read). Leave Cinzel (`--font-display`) and Uncial (`h1.title`) untouched. After the
  change, grep `IM Fell` to confirm only fallback-stack references remain.
- **Patterns to follow:** `--font-body` is the readable stack to mirror.
- **Test scenarios** (in-browser): tale-so-far renders upright EB Garamond; the COMPLETED
  count "N quests tucked away" renders upright EB Garamond; spot-check `.all-complete-note`
  / `.partial-flag` / guildmates — EB Garamond; headings + hero title unchanged.
- **Verification:** `node --check` SYNTAX_OK; readability pass on tale + count + 2 other spots.

### U3. Remove the in-body "End Reward" control

- **Goal:** The `.reward-controls` End Reward label + pill are gone; header pill still works.
- **Requirements:** R5 (cites KTD4)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — render markup `.reward-controls` / "End Reward"
  / `.reward-btn` (~14549-14551), `wireCard` reward-btn handler(s) (~14818, ~14834), CSS
  `.reward-controls` / `.reward-btn` (~2540-2578).
- **Approach:** Remove the `.reward-controls` block from the body template, the matching
  `wireCard` `.reward-btn` listener(s), and the orphaned CSS. Do not touch the header
  `.reward-pill` (render ~14517, CSS ~1180). Confirm `ss.reward` still toggles via the
  header pill after removal.
- **Patterns to follow:** the header `.reward-pill` render + handler is the surviving toggle.
- **Test scenarios** (in-browser): expanded saga + non-saga bodies show no End Reward
  control; header pill still toggles banked ↔ incomplete; no console errors (dead-handler
  check).
- **Verification:** `node --check` SYNTAX_OK; in-browser expand + header-pill toggle, clean console.

### U4. Wrap quest-row pills cleanly on narrow widths

- **Goal:** Pills wrap to a second line inside the card instead of overflowing.
- **Requirements:** R6
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — CSS `.completion-controls` (the quest-row pill
  container).
- **Approach:** Add `flex-wrap: wrap` + a `row-gap` to `.completion-controls` so the
  First-Time-Reaper / Complete-for-Saga / Shard-or-VIP pills wrap within the card.
- **Patterns to follow:** existing pill sizing; keep the wrap gap consistent with the
  inter-pill gap.
- **Test scenarios** (in-browser): at ~420px card width the three pills wrap to two lines
  inside the card, none spilling past the right edge; at wide width they stay on one line.
- **Verification:** `node --check` SYNTAX_OK; narrow in-browser check.

### U5. Pronounced themed zebra striping on quest rows

- **Goal:** Alternating parchment bands separate quests in the active list and the
  completed stash.
- **Requirements:** R7 (cites KTD5)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — CSS on `.quest-block` (targets both active lists
  and `.completed-rollup-body` quests).
- **Approach:** Alternate `.quest-block` backgrounds with a pronounced but themed contrast
  — even ≈ `rgba(101,78,40,0.30)` (warm dark parchment) with a faint inset outline, odd ≈
  `rgba(255,247,220,0.16)` (warm light) — plus a small border-radius. Because completed
  quests are `.quest-block` inside `.completed-rollup-body`, the same rule bands them.
  Use a nth-selector that yields clean per-container alternation (validate parity in the
  browser; adjust `nth-of-type` vs `nth-child` if arcs interleave non-quest siblings).
- **Patterns to follow:** the parchment token palette (`--parchment-*`) for on-theme tints.
- **Test scenarios** (in-browser): consecutive quests in an arc alternate clearly; expanding
  the COMPLETED stash shows its quests banded the same way; banding reads on both light and
  dark color themes.
- **Verification:** `node --check` SYNTAX_OK; in-browser active list + expanded completed stash.

### U6. Redesign the filters container (bounded uniform subcontainers, fold intact, Expand All removed)

- **Goal:** Filter sections are bounded, uniform-sized panels; the fold still collapses
  everything; Expand All is gone with its code cleaned up.
- **Requirements:** R8 (cites KTD6)
- **Dependencies:** none
- **Files:** `sooks-saga-scroll-*.html` — filter markup/render for `#controlsPanel` /
  `.ctl-row*` / `.control-group` (search "controlsPanel" / "ctl-row" / "control-group"),
  the fold logic + `.controls.folded` CSS, the Expand All button markup + its click handler
  + any `state.expandAll` references, and `.control-group` CSS.
- **Approach:**
  - **Panels:** give each `.control-group` a bounded look (subtle fill, thin border, radius,
    padding, centered content); the leading `<label>` header gets a divider (`border-bottom`)
    so it caps its group.
  - **Uniform 2×2:** lay Group By, Search, Audio Alerts, Quest Autocomplete into one grid
    (`repeat(2, minmax(0,1fr))`, equal `min-height`) so all four are the same size; options
    wrap within each panel (`.audio-checks`, `#autocChecks` → `flex-wrap: wrap`). **This
    grid MUST live inside the foldable region** — keep it within a `.ctl-row` (or extend the
    fold's hide rule to the grid) so `.controls.folded` collapses it (a preview `display:grid
    !important` outside the fold mechanism kept it visible when folded — do not repeat that).
  - **Tier:** full-width `.control-group.tier-group` panel holding the tier buttons.
  - **Expand All:** remove the button markup, its click handler, and `state.expandAll`
    wiring/reads; delete its CSS. Confirm nothing else depends on `state.expandAll` (grep;
    if the "expand all quests" capability is referenced elsewhere, remove only the button's
    path and note any residual).
  - **Alignment:** center panel contents (per the user's "left or center — clean, symmetry").
- **Patterns to follow:** existing `.ctl-row` structure and the fold toggle (`.ctl-fold` →
  `.controls.folded`).
- **Test scenarios** (in-browser): four panels are equal-sized with audio checkboxes wrapping
  inside their panel; Tier panel full-width; **clicking the fold bar collapses the ENTIRE
  container to the slim bar and re-opens it**; no Expand All button anywhere; grouping/search/
  audio/autocomplete controls all still function (select, type, check).
- **Verification:** `node --check` SYNTAX_OK; in-browser fold/unfold cycle + each control works;
  grep confirms no dangling `expandAll` / Expand All references.

### U7. Render the collapsed COMPLETED rollup as a fractional completion progress bar

- **Goal:** The collapsed COMPLETED bar reads as a green→gold progress bar filled to
  completed ÷ total, with bold black legible text.
- **Requirements:** R9 (cites KTD7)
- **Dependencies:** U2 (the count font), U5 (the stash it caps)
- **Files:** `sooks-saga-scroll-*.html` — the completed-rollup render (`.completed-rollup` /
  `.completed-rollup-header` / `.completed-rollup-count` / `.completed-rollup-check`) and its
  CSS (~2100-2130).
- **Approach:** At render time compute the saga's completion fraction from `sagaStatus`
  (`done / total`) — the same source the header Complete bar uses — and drive the bar's fill
  to that percent (e.g. a per-element `--fill` custom property set inline, or a fill child
  sized to the percent). Style `.completed-rollup-header` as a completion bar: green→gold
  gradient fill up to `--fill`, empty parchment track after it, a crisp dark leading edge at
  the boundary, gold border + soft glow (mirroring the saga `.all-done`/`.has-progress` bar),
  and **bold black text** (`font-weight: 800; color: #000`) with a faint light halo on the
  count for legibility across the fill/track. Keep the same bar height as today. Applies to
  saga and non-saga containers.
- **Technical design (directional):** fill via
  `linear-gradient(90deg, green 0%, gold calc(var(--fill) - 12%), gold var(--fill), track
  var(--fill), track 100%)` with a 2px dark rule positioned at `left: var(--fill)`. Not
  implementation-binding — a fill child element sized to the percent is equally acceptable.
- **Patterns to follow:** the saga completion bar (`.saga-progress-bar.all-done` /
  `.has-progress` fill + border + glow) and how the header bar computes its width from
  `sagaStatus`.
- **Test scenarios** (in-browser): a saga with 5/10 completed shows a ~50% green→gold fill
  with a clear empty track and a crisp boundary; a saga with all completed shows a full bar;
  the "✓ COMPLETED" + "N quests tucked away — click to show" text is bold black and readable
  over both fill and track; clicking still expands/collapses the stash; a non-saga container
  behaves the same.
- **Verification:** `node --check` SYNTAX_OK; in-browser at partial and full completion.

---

## Scope Boundaries

**In scope:** U1–U7.

**Out of scope / unchanged:** Cinzel headings, the Uncial hero `h1.title`, the color theme,
the progress-bar visuals from build `07192026-1`, the reaper/completion header bars' own
styling (only their row placement changes in U1). No new fonts. The "expand all quests"
capability is removed by user request (U6), not preserved elsewhere.

---

## Verification Contract

- **Syntax:** `node --check` on the extracted `<script>` reports `SYNTAX_OK`.
- **Visual:** browser automation works on this project (localhost + Claude-in-Chrome). Every
  unit's visual behavior is verified **by Claude** on a crafted test character (inject a
  character with progress + a fake LFM, `state.expandAll`/expand as needed, exercise fold and
  the completed rollup), then cleaned up (`localStorage.removeItem('sooksSagaScroll')`, remove
  the test character, stop the server, close the tab). All seven designs were already
  prototyped in-browser during planning; the build pass re-confirms them on the real edits.
- **No schema change:** state stays v14; `sooksSagaScroll` storage untouched.

## Definition of Done

- U1–U7 implemented in a copied-forward build file (copy `07192026-1` first; never edit the
  parked build in place).
- Build version stamped (mmddyyyy.x) per the build-versioning skill.
- `node --check` passes; in-browser pass confirms: header stack + LFM panel + narrow snap +
  pill alignment; no IM Fell in reading text; End Reward control gone + header pill works;
  narrow pill wrapping; pronounced zebra (active + completed stash); filters uniform panels
  with a working fold and no Expand All; and the completed rollup as a fractional progress
  bar with bold black text.
- README "Currently parked" line + build note updated; park retention applied (2 most recent
  + cross-day anchor) per garage-park-retention.
