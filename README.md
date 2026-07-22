# Sook's Saga Scroll

**Category:** Personal

> **Currently parked:** `sooks-saga-scroll-07222026-1.html` — Build
> `07222026.1` (2026-07-22). 2 most recent overall:
> `sooks-saga-scroll-07222026-1.html` + `sooks-saga-scroll-07192026-5.html`.
> The 2nd-most-recent (`-07192026-5`, 07-19) is itself the most-recent
> previous-day build, so it doubles as the cross-day rollback anchor — no
> distinct 3rd slot needed; retain set is **2 files**. Pruned this park:
> `sooks-saga-scroll-07192026-4.html` + `sooks-saga-scroll-07182026-9.html`
> → `_to_delete/`. **Update 81 — Terror of Demogorgon (data-only; no schema
> change, still v14):** added DDO's 10th expansion (Underdark, released
> 2026-07-22) as **two containers** — `h-demogorgon` (Heroic **L11**) and
> `l-demogorgon` (Legendary **L37**) — each holding the same **12 quests** in 3
> arcs (Part One ×5 / Part Two ×5 / Side ×2). **Level-ordered placement**
> (verified): `h-demogorgon` sits **between `h-mist` (L10-12) and `h-myth-drannor`
> (L13)**; `l-demogorgon` sits **at the bottom of the Legendary group after
> `l-chill` (L36)** (intra-group order is array order — no render-time level
> sort). New `SAGAS` / `SAGA_STORIES` / `QUEST_GIVERS` entries + a single
> `QUEST_DETAILS["h-demogorgon"]` block (12 quests, sourced named-monster arrays);
> `l-demogorgon` resolves all 12 details via the cross-tier `lookupQuestDetails()`
> fallback. Verified in-browser (localhost + Claude-in-Chrome): data self-check
> **0 errors**, **64 containers / 541 quests / 100% detailed**, legendary fallback
> **12/12**, both groups ascending by level, no console errors. **Preview-data
> gaps flagged for U6 live re-source** (`TODO(u81-resource)` markers): 4
> placeholder synopses (Hideous to Behold, For Want of a Heart, The Fetid Wedding,
> Stealing from Sorcere), 2 placeholder locations, the Grin Ousttyl/Ousstyl giver
> spelling, and full trash-mob monster enumeration. Update this line on every park.

> **Publishing (GitHub Pages) — paused 2026-07-19 on a GitHub Actions incident.**
> Repo `eddiefiggie/sooks-saga-scroll` is live and Pages is configured in
> **branch-deploy** mode (`main` `/`), serving `index.html` (a copy of the parked
> build) at **https://eddiefiggie.github.io/sooks-saga-scroll/**. The first build
> was blocked because GitHub Actions was `degraded_performance` (runners wouldn't
> provision — both the Actions workflow and the branch-deploy build stalled). This
> is a transient GitHub outage, not a config problem. **To finish once Actions is
> green (check githubstatus.com):** `gh api -X POST repos/eddiefiggie/sooks-saga-scroll/pages/builds`
> (or push any commit), then the site goes live. `.github/workflows/deploy.yml`
> (auto-derives `index.html` from the "Currently parked" line above) is committed
> for later — to use it instead of manual copies, switch the Pages source back to
> GitHub Actions (`gh api -X PUT .../pages -f build_type=workflow`).

A single-file, offline-capable HTML compendium for tracking Dungeons & Dragons
Online (DDO) saga progress across multiple characters. Parchment-and-wax-seal
aesthetic — Heroic, Epic, and Legendary sagas, per-quest ☠ REAPER badges,
banked vs. redeemed reward states, and DDO Wiki–sourced story / monster / trap
details tucked behind each quest. Companion to the source spreadsheet
"Sook's Saga Database" (Google Sheets).

## Files
- `sooks-saga-scroll-07222026-1.html` — the live build. Open in any
  browser; no server or build step. Progress auto-saves to browser
  storage (key `sooksSagaScroll`, **schema v14** as of Build
  07142026.1 — see notes below). See **Data Schema & Import/Export
  Contract** below for the durable shape and the rules every future
  build must follow.

  **Build 07222026.1 — Update 81 "Terror of Demogorgon" (data-only; no schema
  change, still v14).** Added DDO's 10th expansion (Underdark, released
  2026-07-22) as two containers: `h-demogorgon` (Heroic L11, giver Haruhak
  Deepgift) and `l-demogorgon` (Legendary L37, giver Eramura Orefinder), each 12
  quests in 3 arcs (Part One ×5 / Part Two ×5 / Side ×2). Level-ordered:
  `h-demogorgon` between `h-mist` (L10-12) and `h-myth-drannor` (L13);
  `l-demogorgon` last in the Legendary group after `l-chill` (L36). New `SAGAS` /
  `SAGA_STORIES` / `QUEST_GIVERS` + a single `QUEST_DETAILS["h-demogorgon"]` block
  (12 quests, sourced named-monster arrays); `l-demogorgon` resolves details via
  the cross-tier `lookupQuestDetails()` fallback (12/12, verified). Self-check
  62 → 64 containers / 517 → 541 quests, 100% detailed, 0 errors. All content
  ddowiki-sourced (U81 dossier in `data/`); 4 preview placeholder synopses +
  2 locations + the Grin Ousttyl/Ousstyl spelling + full monster enumeration are
  flagged `TODO(u81-resource)` for the U6 live re-source pass (not yet run).

  **Build 07192026.5 — multi-LFM pill + filter checkbox alignment (CSS + JS; no
  schema change, still v14).** **(1) Header LFM pill scales with group count.**
  `lfmForTier`'s aggregate now exposes `groups` — the full tier-filtered list
  sorted by skulls — and `renderSagaCard` collects every group across the saga's
  quests into `lfmGroups`. The fixed 210×72 pill then branches: **1** → the
  3-row pill (unchanged); **2–3** → stacked `.lfm-mini` rows (a tier-colored left
  stripe + `☠ R<n>` + leader, focused on level + who); **4+** → `.lfm-frac`, a
  segmented color bar (one `.lfm-seg` per group, tinted low-gold / mid-orange /
  high-red) with an `N groups` count and the top group. A `length >= 2 && <= 3`
  guard keeps a zero-group saga from rendering an empty pill (the first pass used
  `<= 3`, which caught 0). A **code-review pass** then caught a second boundary
  miss: the collected list included 0-skull (non-reaper) groups, so a
  normal-difficulty LFM rendered a misleading `☠ R0` pill — fixed by filtering the
  collected groups to `skulls > 0`, restoring build .3's declared-reaper-only gate. **(2) Filter checkbox/radio alignment.** `.audio-checks`
  (shared by Audio Alerts and Quest Autocomplete) changed from a centered
  `flex-wrap` row to a `flex-direction: column; align-items: flex-start`, so every
  checkbox/radio left-aligns in a vertical column; the `.autoc-divider` became a
  horizontal rule to suit the column. **Verification: in-browser** (localhost +
  Claude-in-Chrome) — injected 1/2/3/5-group sagas render the four variants
  correctly; **0 empty pills**; Audio checkmarks all share one x and Autocomplete
  checkmarks all share one x (aligned columns); at 380px no search overflow, no
  page horizontal scroll, header pill stays within its band; no console errors.
  Live file: `sooks-saga-scroll-07192026-5.html`. Park retention: kept
  `sooks-saga-scroll-07192026-5.html` + `sooks-saga-scroll-07192026-4.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07182026-9.html`
  (3 files); pruned `sooks-saga-scroll-07192026-3.html` → `_to_delete/`.

  **Build 07192026.4 — LFM pill redesign + quest-row layout batch (CSS + JS; no
  schema change, still v14).** Six changes, each prototyped and validated from a
  live in-browser draft. **(1) Header LFM pill → fixed-size 3-row.** The
  saga/non-saga header LFM `.lfm-skull-tag` is now a fixed 210×72 three-row pill:
  row 1 `☠ R<n> · <fill>/6`, row 2 the leader, row 3 the zone — same footprint on
  every card (`.hdr-t3 .lfm-r1/.lfm-r2/.lfm-r3`). **(2) Pills vertically aligned.**
  `.saga-header.has-lfm` reserves a fixed right band (`minmax(0,1fr) 248px`) and
  centers the pill in it, so every header pill shares one x. **(3) Row 3 = zone.**
  A new `_questAreaIdMap` (quest_id → area_id, built in `_setQuestMaps`) plus the
  areas table resolves each LFM's quest to its **adventure area**; `refreshLfmIndex`
  stores `zone` on the group record and `lfmForTier`'s aggregate carries it.
  Reliable (always resolves) and distinct from the quest name (*The Haunting of
  Saltmarsh* → *The Haunted House*). Wired to the quest's area rather than the LFM
  leader's live location — no live LFMs were up to confirm the leader exposes a
  location field, and an unverifiable source would silently render an empty row.
  **(4) Reaper-tier trackers color-matched.** The right-side Low/Mid/High Reaper
  corner captions (`th.gm-fold.low/.mid/.high`, tint from `REAPER_CARDS`) now carry
  the same neutral-gold / orange / red as the LFM pill tiers. **(5) "Given by"
  wraps.** `.quest-giver` changed from `white-space:nowrap; flex-shrink:0` to
  `normal` + `overflow-wrap:anywhere` + `min-width:0`, so a long quest-giver rolls
  under instead of spilling past the card edge. **(6) Quest-row pills + LFM align.**
  The Reaper/Saga/Skip pills are wrapped in `.q-actions` (a horizontal row that
  stacks into an aligned vertical column only below ~490px container width — a
  plain flex row, *not* an `auto-fit` grid, which collapsed at every width), and
  the LFM `.quest-live` left edge aligns to the pills via
  `margin-left: calc(var(--q-check) + 20px)` (derived from the checkbox width, not
  a magic number). **Also folded in:** the filters **Search field** overflow —
  `#questSearch` had `min-width: var(--ctl-w)` which beat `max-width:100%` and
  spilled the input out of its 2×2 sub-panel at the narrowest widths; now
  `min-width:0`. **Verification: in-browser** (localhost + Claude-in-Chrome) —
  header pills uniform 210×72 and left-aligned with real resolved zones; quest
  pills row-at-wide / aligned-column-at-narrow with the LFM tracking their left
  edge; no Search overflow and no page horizontal scroll at 380px; zone wiring
  confirmed end-to-end against the live areas table (655 quests); no console
  errors. Live file: `sooks-saga-scroll-07192026-4.html`. Park retention: kept
  `sooks-saga-scroll-07192026-4.html` + `sooks-saga-scroll-07192026-3.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07182026-9.html`
  (3 files); pruned `sooks-saga-scroll-07192026-2.html` → `_to_delete/`.

  **Build 07192026.3 — header corner-widget symmetry (CSS + 1-line JS; no
  schema change, still v14).** Fixes the reported bug where, as the window
  narrows, the upper-left (Server/Guild) and upper-right (Characters/LFM)
  fixed clusters *grew* and slid over the center. Root cause: the `≤1179px`
  media block bumped `.corner-panel` from 200px to **320px** (growth-on-narrow)
  and the reflow fired at a hard `1179px` breakpoint rather than at real
  collision. Fix, planned in
  `docs/plans/2026-07-19-003-fix-header-corner-symmetry-plan.md`: a single
  `:root { --corner-w: clamp(150px, calc((100vw - 688px)/2 - 26px), 200px) }`
  drives both the corner width (`.corner-panel { width: var(--corner-w) }`, so
  left == right by construction) and the center gutter (`.scroll { max-width:
  min(1150px, calc(100vw - 2*var(--corner-w) - 52px)) }`), collapsing the old
  `1180–1620` special band into one continuous fixed regime. Panels hold 200px,
  then **shrink symmetrically toward the 150px floor**, staying pinned to both
  edges until ~1040px, where they reflow into the existing `#narrowStack` single
  column (the `320px` growth rule is deleted). The reflow breakpoint moved
  `1179 → 1040` in the `@media` query **and** the `initNarrowStack` `matchMedia`
  (kept in sync; the value is duplicated by design, noted in-code). **Verified:**
  in-browser (localhost + Claude-in-Chrome) — at 1680/1269px both clusters
  measure 200px, equal, pinned to both edges (center parchment centered at
  1150px); width is monotonically non-increasing as the window narrows (no
  growth); at narrow widths the clusters form a clean centered single column.
  (Known pre-existing, not from this change: on classic-scrollbar systems the
  right cluster sits ~15px further from the edge — the vertical scrollbar's
  width; a `scrollbar-gutter` compensation is a possible follow-up.) Live file:
  `sooks-saga-scroll-07192026-3.html`. Park retention: kept
  `sooks-saga-scroll-07192026-3.html` + `sooks-saga-scroll-07192026-2.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07182026-9.html`
  (3 files); pruned `sooks-saga-scroll-07192026-1.html` → `_to_delete/`.

  **Build 07192026.2 — header/LFM layout + font sweep + filters redesign +
  zebra + completed progress bar (CSS + JS markup; no schema change, still
  v14).** A seven-unit UI batch, planned in
  `docs/plans/2026-07-19-002-feat-header-stack-font-sweep-end-reward-plan.md`,
  with every unit prototyped and approved from a live in-browser draft during
  planning. **(U1) Header restructure.** `.saga-header` is now a 2-column grid
  (`"t1 t3"` / `"t2 t3"`): the level pill + title get row 1, the reward pill +
  both progress bars drop to row 2, and the LFM notification fills the reclaimed
  right-side space spanning both rows. Below the `@container` breakpoint it
  restacks to `"t1"/"t2"/"t3"` with the LFM as a full-width bottom row. The
  reward pill bottom-aligns with the bars (`align-items:flex-end`). Applies to
  saga and non-saga containers. **(U2) IM Fell retired from reading text.**
  `--font-flavor` now points at the EB Garamond stack (sweeping all ~9
  consumers); `.saga-story-body` (tale so far) and `.completed-rollup-count` are
  forced upright EB Garamond (the italic + tiny + faded combo was the real
  readability killer). Cinzel headings and the Uncial `h1` title are untouched.
  **(U3) End Reward control removed.** The in-body `.reward-controls` label +
  `.reward-btn` pill, its `wireCard` handler, and its CSS are gone; the header
  `.reward-pill` is the single banked/incomplete toggle. **(U4) Pill wrapping.**
  `.completion-controls` flex-wraps so quest-row pills wrap inside the card on
  narrow widths instead of spilling out. **(U5) Quest zebra.** Alternating warm
  parchment tints on `.quest-block` (pronounced) separate rows in the active
  list and the completed stash. **(U6) Filters redesign.** Each section is a
  bounded, centered panel; Group by / Search / Audio Alerts / Quest Autocomplete
  are a uniform 2×2 grid (`.ctl-row-top`) with options wrapping inside; Tier is a
  full-width panel below. The fold still collapses the whole container (the grid
  uses no `!important`, so `.controls.folded .ctl-row { display:none }` wins).
  The **Expand all** button was removed and its legacy fully cleaned out — the
  button, its handler, `state.expandAll` (default/META_KEYS/load), and the
  `collapsed` set are all deleted; per-saga expand lives entirely in `expanded`.
  **(U7) Completed rollup as a progress bar.** The collapsed COMPLETED bar is a
  completion progress bar filled to the saga's `done / total` (from `sagaStatus`,
  the same source as the header Complete bar) via an inline `--fill`; green→gold
  fill, empty parchment track, crisp dark leading edge at the boundary, gold
  trim, and bold black text. **Verification: `node --check` `SYNTAX_OK`;** clean
  boot + render (no console errors); **in-browser visual pass by Claude** on a
  crafted test character confirmed all seven units at wide + narrow widths —
  header stack / LFM right-panel / narrow snap / pill alignment; readable tale +
  count fonts; End Reward gone with the header pill still working; pills wrapping
  on narrow; pronounced zebra in the active list AND the completed stash; the
  filters as uniform panels with **the fold collapsing the whole container** and
  no Expand All; and the completed rollup as a ~50%-filled green→gold progress
  bar with bold black text. Live file: `sooks-saga-scroll-07192026-2.html`. Park
  retention: kept `sooks-saga-scroll-07192026-2.html` +
  `sooks-saga-scroll-07192026-1.html` (2 most recent) + cross-day anchor
  `sooks-saga-scroll-07182026-9.html` (3 files); nothing pruned (only 3 builds
  exist).

  **Build 07192026.1 — autocomplete reaper fix + DS&A CPU pass + progress-bar
  visuals (CSS + JS; no schema change, still v14).** A five-item batch, planned
  in `docs/plans/2026-07-19-001-feat-autocomplete-fix-cpu-and-bar-visuals-plan.md`.
  **(A) Reaper autocomplete re-fire bug.** First-Time Reaper is a permanent,
  one-time claim. An already-reapered quest whose completion seal was later
  cleared (saga redeemed → new cycle) used to, on a reaper-mode autocomplete
  re-run, re-add to `reaperPopQueue` and toast "First-Time Reaper sealed
  automatically" — celebrating an FTR that was already banked. `_acApply` now
  computes `wantReaper = mode==="reaper" && !NO_REAPER.has(q) && !alreadyReaper`,
  so that case degrades to a plain completion (green seal + cascade + auto-bank,
  normal "Quest Completed" toast, **no** ☠ pop); the fully-done case still
  no-ops. **(B) DS&A CPU pass — byte-identical output.** `renderSagaList()`
  rebuilds all ~53 cards / ~479 quest rows on every click/autocomplete/poll, so
  per-row linear scans dominated. Added load-time indexes over the static
  `SAGAS`/`QUEST_DETAILS` tables (no runtime invalidation): `SAGA_BY_ID` (kills
  `SAGAS.find` at 8 hot sites), `QUEST_DETAILS` exact + normalized maps (kills
  `lookupQuestDetails`'s two fallback scans), and a normalized-quest→saga index
  (kills the `_sagaForQuest` scan). `lfmForTier` is memoized per `_lfmIndex`
  generation (it ran ~2× per quest per render); `sagaStatus` folds three
  `.filter` passes into one loop; the `_lfmIndex` change-detector reuses a
  cached signature (object-identity–keyed, self-healing on reset) instead of
  re-serializing the index every poll. **(C) LFM bar glow removed.** The
  saga-header `.lfm-skull-tag` indicator already glows; the container progress
  bar also glowed via `.lfm-mid`/`.lfm-high` (border + wash + pulse). The bar no
  longer receives `lfmSkullClass` and those dead CSS rules are removed — the
  indicator is the sole LFM signal (`@keyframes lfmPulse` kept; still used by the
  skull-tag and quest-row pulses). **(D) Reaper leading-edge contrast.** The
  partial reaper fill gets a crisp dark leading edge (inset `--wax-red-dark`
  right-side shadow) so the fill/track boundary reads sharply instead of washing
  into the light parchment as it advances L→R. **(E) Flat partial bars.**
  Partial (has-progress, not complete) bars are now flat solid fills — solid
  green for completion, solid wax-red for reaper — with no gradient and no shine
  band; only complete bars (`.all-done` / `.reaper.sealed`) keep the gradient +
  shimmer. **Deferred:** the structural single-card re-render (DS&A finding #4 —
  re-render only the toggled card instead of the whole list) is parked to its own
  build; it needs a full-interaction visual pass. **Verification (by Claude):
  `node --check` `SYNTAX_OK`** (node is installed on this machine now — the older
  "no node, use `jsc`" note is stale); clean boot + render with no console
  errors; **in-browser visual pass via localhost + Claude-in-Chrome** on a
  crafted test character confirmed C (LFM'd saga: ☠ R8 indicator glows, its
  completion bar stays plain green — no bar glow), D (reaper partial bars carry a
  crisp dark leading edge at both ~30% and ~15% fill), and E (partial completion
  + reaper bars are flat solid; the fully-complete 10/10 bars keep the green→gold
  / wax-red→gold gradient); a **behavioral `_acApply` test** confirmed A across
  all three cases (fresh reaper → `reaper:true` + pop; already-reapered with
  cleared seal → `reaper:false`, no pop, green seal set, FTR preserved;
  fully-done → null no-op). The DS&A units (B) are output-preserving.
  Live file: `sooks-saga-scroll-07192026-1.html`. Park retention: kept
  `sooks-saga-scroll-07192026-1.html` + `sooks-saga-scroll-07182026-9.html`
  (2 most recent; .9 doubles as the cross-day anchor); pruned
  `sooks-saga-scroll-07182026-8.html` and `sooks-saga-scroll-07142026-2.html`
  to `_to_delete/` on 2026-07-19.

  **Build 07182026.9 — CPU/animation performance pass (CSS only; no schema
  change, still v14).** The app ran Chrome hot because every progress bar with
  *any* progress ran a continuous `background-position` "sheen" animation —
  ~121 of them at once after the twin-bars change, each forcing a raster repaint
  per frame. **(1)** The traveling shimmer now runs **only on complete bars** —
  the completion bar when `.all-done`, the reaper bar when `.sealed` (fully
  reaped) — and animates a GPU-composited `transform` instead of
  `background-position`; partial `has-progress` bars get a **static gold shine**
  (no animation). **(2)** The data-scaling box-shadow pulse glows (lfm-mid/high
  container bars, high LFM tag, high LFM quest row, first-time-reaper row) now
  pulse a **GPU-composited `opacity` overlay** instead of animating `box-shadow`;
  the lfm-mid/high bar glows are anchored on a non-clipping wrapper because the
  bar itself has `overflow: hidden` (a child's outer glow would be clipped).
  Reduced-motion behavior is preserved per rule (ftrRowPulse keeps its distinct
  `@media (reduce)` resting glow). Singleton chrome pulses (sagaNow/add/flow —
  one instance each) left as-is. **Verification: `node --check` (`SYNTAX_OK`) +
  before/after in-browser measurement by Claude** — running sheen animations
  dropped **121 → 8** with a fully-partial test character (only the 8 complete
  bars animate, and they animate `transform`, not `background-position`); all
  four converted pulses confirmed to animate `opacity` with zero running
  `box-shadow` animations; visual split, `.5`–`.8` regression, and clean console
  confirmed by screenshot. Plan:
  `docs/plans/2026-07-18-003-perf-reduce-cpu-animation-load-plan.md`. Live file:
  `sooks-saga-scroll-07182026-9.html`. Park retention: kept
  `sooks-saga-scroll-07182026-9.html` + `sooks-saga-scroll-07182026-8.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07142026-2.html`
  (3 files); pruned `sooks-saga-scroll-07182026-7.html` on 2026-07-18.

  **Build 07182026.8 — pill + LFM polish (CSS only; no schema change,
  still v14).** (superseded by 07182026.9 — CPU/animation performance pass.)
  (1) The reward pill's **Banked and Incomplete states are now
  the same width** — the pill min-width was raised (~116px) to fit the wider
  "INCOMPLETE" label so "BANKED" pads up to match instead of being narrower.
  (2) The **LFM notice stands out more**: the container header's neutral
  (R1-R3 / undeclared) `.lfm-skull-tag` state gets a gold accent border +
  stronger fill + soft glow (it previously near-vanished into the parchment;
  the orange `.mid` / red `.high` tier states are unchanged), and the quest
  row's live-LFM line (`.quest-live`) gets bolder text, a stronger fill, a
  thicker left rule, and a subtle lift. **Verification: `node --check`
  (`SYNTAX_OK`) + in-browser visual pass by Claude** — loaded the build,
  injected a test character and faked LFM data, and confirmed at zoom that the
  pills are equal width and both LFM surfaces stand out. Live file:
  `sooks-saga-scroll-07182026-8.html`. Park retention: kept
  `sooks-saga-scroll-07182026-8.html` + `sooks-saga-scroll-07182026-7.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07142026-2.html`
  (3 files); pruned `sooks-saga-scroll-07182026-6.html` on 2026-07-18.

  **Build 07182026.7 — two visual-pass fixes (CSS only; no schema change,
  still v14).** (superseded by 07182026.8 — pill uniform-width + LFM standout.)
  From the user's eyeball of .6: **(1)** the reward pill
  (Banked/Incomplete) floated *above* the progress bars because the header
  actions row centered it against the taller caption+bar column — bottom-aligned
  the actions row (`align-items: flex-end`) and set the pill to the bar height
  (22px), so the pill and both bars are the same size and line up top-to-bottom.
  **(2)** the reaper bar's count numbers were dark text on the dark-red fill (no
  contrast, worse as reaper progress advances) — the reaper count is now cream
  text with a dark halo, legible over both the filled (dark-red) and unfilled
  (light) portions at any progress level (completion bar text unchanged — its
  green→gold fill reads fine dark). **Verification: `jsc` syntax (`SYNTAX_OK`)
  + user visual pass — approved in a real browser.** Live file:
  `sooks-saga-scroll-07182026-7.html`. Park retention: kept
  `sooks-saga-scroll-07182026-7.html` + `sooks-saga-scroll-07182026-6.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07142026-2.html`
  (3 files); pruned `sooks-saga-scroll-07182026-5.html` on 2026-07-18.

  **Build 07182026.6 — header/level polish (CSS + JS markup; no schema
  change, still v14).** (superseded by 07182026.7 — reward-pill alignment +
  reaper-text contrast fixes.) Three user-requested tweaks on top of .5: (1) the
  container header's **LFM notice is now a full-width, centered bar** on its
  own row (was a small right-justified chip) — it stretches across the header
  with left/right padding and centered content. (2) The **level tag moved to
  the LEFT of the name** on both saga headers and quest rows. (3) **Level tags
  are uniform width** (saga min 78px, quest min 50px, centered) so shorter
  labels pad to match longer ones. Retains .5's working roll-over.
  **Verification: `jsc` syntax (`SYNTAX_OK`) + static only — still needs a
  real-browser visual pass.** Live file: `sooks-saga-scroll-07182026-6.html`.
  Park retention: kept `sooks-saga-scroll-07182026-6.html` +
  `sooks-saga-scroll-07182026-5.html` (2 most recent) + cross-day anchor
  `sooks-saga-scroll-07142026-2.html` (3 files); pruned
  `sooks-saga-scroll-07182026-4.html` on 2026-07-18.

  **Build 07182026.5 — code-review fixes for the roll-over (CSS + JS
  markup; no schema change, still v14).** (superseded by 07182026.6 — the
  container-query fix carries forward; .6 adds LFM-bar + level-tag polish.) A `ce-code-review` pass on .4
  caught three issues: **(1) CRITICAL** — .4's container queries never
  fired because `container-type: inline-size` sat on the *same* element
  (`.saga-header` / `.quest`) the `@container` rules targeted, and a
  container queries its *descendants*, not itself — so the narrow/stacked
  layouts were dead code and nothing rolled. Fixed by moving `container-type`
  up to the wrappers (`.saga` card for the header, `.quest-block` for the
  quest row); the `@container` rules stay on `.saga-header` / `.quest`, now
  descendants, so rolling actually works. **(2)** Restored the quest DOM to
  controls-before-name so keyboard tab order is linear again (grid still
  paints name-on-top when narrow). **(3)** Raised the quest roll threshold
  560→640px so a full control cluster doesn't crush the quest name into a
  sliver just above the breakpoint. The review also confirmed the fix is
  safe (no containment side-effects: `.ac-toast` / `.data-corner` aren't
  inside cards; `.saga` is already `position:relative`). **Verification:
  `jsc` syntax (`SYNTAX_OK`) + static + a two-reviewer code review — still
  needs a real-browser visual pass** (now the rolling should actually
  function; breakpoints 470/640px are tunable). Live file:
  `sooks-saga-scroll-07182026-5.html`. Park retention: kept
  `sooks-saga-scroll-07182026-5.html` + `sooks-saga-scroll-07182026-4.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07142026-2.html`
  (3 files); pruned `sooks-saga-scroll-07182026-3.html` on 2026-07-18.

  **Build 07182026.4 — roll-over UNIFORMITY fix (CSS only; no schema
  change, still v14).** (superseded by 07182026.5 — .4's container queries
  were on the wrong element and never fired; .5 fixes it.) User feedback on .3: different saga containers
  rolled at different widths (some cards stacked, others didn't) and the
  LFM tag sometimes shared a row. Root cause: .3 rolled tiers with
  `flex-wrap`, which wraps based on CONTENT — so a card with a live LFM or
  a long name wrapped at a different width than one without. Fix: both the
  saga header and the quest row now use **CSS `grid-template-areas`**
  switched purely by the row's own container-query width, with **no
  content-based wrapping** — so every same-width row rolls identically and
  in lockstep (uniform + proportional). Per user request the **LFM now
  always occupies its own dedicated row** (never shares a line), auto-placed
  full-width and **right-justified** on the container header; because it has
  its own row it no longer shifts the pill+bars, so those columns align
  across all cards regardless of LFM. States: wide = name+actions on row 1,
  LFM on its own row; narrow (header ≤470px / quest ≤560px container width) =
  fully stacked. All other .3 behavior (shiny reaper bar, decoupled LFM,
  ≤1179px `#narrowStack` coexistence) retained. **Verification: `jsc`
  syntax (`SYNTAX_OK`) + static only — the responsive/visual behavior still
  needs a real-browser pass** (breakpoints 470/560px are tunable knobs).
  Live file: `sooks-saga-scroll-07182026-4.html`. Park retention: kept
  `sooks-saga-scroll-07182026-4.html` + `sooks-saga-scroll-07182026-3.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07142026-2.html`
  (3 files); pruned `sooks-saga-scroll-07182026-2.html` on 2026-07-18.

  **Build 07182026.3 — uniform roll-over row stacking (CSS + JS markup;
  no schema change, still v14).** (superseded by 07182026.4 — the .3
  flex-wrap rolled non-uniformly; .4 reworks it to CSS grid.) Built via the Compound Engineering
  pipeline (brainstorm → plan → work → review), plan at
  `docs/plans/2026-07-18-002-feat-uniform-rollover-stacking-plan.md`. Both
  the saga container headers and the quest rows now use one uniform 3-tier
  responsive system that rolls **progressively and atomically** as each row
  runs low on width, driven by **CSS container queries** — each header/row
  is its own `container-type: inline-size` context, so a row rolls on *its*
  width and same-width siblings roll in lockstep (keeping columns aligned).
  Tiers: **T1** name+level (stays top), **T2** the actions group (container:
  reward pill + reaper bar + completion bar; quest row: the completion
  controls), **T3** LFM. Lowest tier (LFM) rolls to its own line first, then
  T2; each tier is a nowrap group so a lone bar/button never strands. Also:
  (a) the container header's LFM tag is **decoupled** from the completion bar
  into its own T3 element; (b) quest rows adopt **B1** — name+level on top,
  completion controls roll below when narrow (`order` keeps controls left at
  wide); (c) fixed-width action columns so the pill/bars align across saga
  cards and the control buttons align across quest rows; (d) items within a
  row are height-aligned; (e) the **reaper bar is now shiny** — it inherits
  the completion bar's gold trim/glow/sheen (behind `prefers-reduced-motion`)
  over a wax-red→gold fill; (f) the old ragged ≤700px header flex-wrap is
  retired; coexists with the ≤1179px `#narrowStack` corner reparent.
  **Verification: JS syntax only (`jsc` — `SYNTAX_OK`) + static checks.** The
  responsive/visual behavior (tier thresholds, roll smoothness, column
  alignment, the shiny reaper look) was **NOT** browser-verified this session
  (automation blocked) and needs a real-browser eyeball; the container-query
  thresholds (header 640/480px, quest 560px) are tunable knobs. Two known
  tune points: the header's T2 (pill+bars) shifts slightly on cards that have
  a live LFM vs. those that don't; and the quest-row wide→narrow control
  reflow (controls move from left-of-name to below-name) may want smoothing.
  Live file: `sooks-saga-scroll-07182026-3.html`. Park retention: kept
  `sooks-saga-scroll-07182026-3.html` + `sooks-saga-scroll-07182026-2.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07142026-2.html`
  (3 files); pruned `sooks-saga-scroll-07182026-1.html` on 2026-07-18.

  **Build 07182026.2 — code-review follow-up: gate the narrow-stack
  "Server" label (CSS only; no schema change, still v14).** (superseded by
  07182026.3) A
  `ce-code-review` adversarial pass on Build .1 flagged that, in the
  narrow priority stack, the Server group's divider + `::before "Server"`
  label rendered even when the left cluster had no visible content — when
  no server is set, `#serverPop` and `#guildCorner` both collapse via
  `:empty { display:none }`, but the label sits on the always-present
  parent, so a labeled header floated over blank space on first run and
  for any freshly-added character. Both the divider and the label are now
  gated on `#narrowStack .data-corner.left:has(.corner-panel:not(:empty))`,
  so they appear only once the cluster has content. Surgical CSS-only
  change from .1 (verified: the `<style>` diff is exactly this gate).
  Verified: script compiles clean under JavaScriptCore (`jsc`). Live file:
  `sooks-saga-scroll-07182026-2.html`. Park retention: kept
  `sooks-saga-scroll-07182026-2.html` + `sooks-saga-scroll-07182026-1.html`
  (2 most recent) + cross-day anchor `sooks-saga-scroll-07142026-2.html`
  (3 files); nothing pruned this park.

  **Build 07182026.1 — responsive priority stack + header twin bars +
  band removal (CSS + JS; no schema change, still v14).** (superseded by
  07182026.2 — narrow-stack "Server" label gating) Built via the
  Compound Engineering pipeline (brainstorm → plan → work); plan at
  `docs/plans/2026-07-18-001-feat-responsive-stack-and-header-bars-plan.md`.
  (1) NARROW-WINDOW PRIORITY STACK — at ≤1179px the two fixed corner
  clusters (Characters + LFM on the right, Server pop + Guild on the left)
  used to reflow into two independent centered wrapping rows with no defined
  order, reading as scrambled. A `matchMedia('(max-width:1179px)')` handler
  now reparents both clusters into a new `#narrowStack` div that sits
  directly below the Filters panel, so the narrow view is one coherent
  labeled column: Filters → Characters → LFM alerts → Server (the Server
  group gets a divider + section label). Above 1179px both clusters are
  moved back to their exact original `<body>` positions, so the desktop
  fixed-corner layout is byte-identical. Nodes are *moved*, never recreated,
  so live-refresh bindings and fold state survive and `#charCorner` /
  `#lowCorner` / `#serverPop` / `#guildCorner` keep their IDs. (2) HEADER
  TWIN BARS — each saga and non-saga container header replaces the old
  two-state `.ftr-badge` + lone progress bar with **two equal side-by-side
  bars on the far right, caption above and count inside**: a wax-red
  First-Time Reaper bar (fill = `reaperDone / reaperEligibleTotal`,
  `NO_REAPER` quests excluded; muted `0/0` when a container has no
  reaper-eligible quests; a gold hairline marks it fully reaped) and the
  re-themed green completion bar (unchanged `st.done / st.total` metric).
  `sagaStatus()` now also returns `reaperEligibleTotal`. The live LFM tier
  tag still rides under the completion bar; count text is clipped so it
  never spills the bar. (3) BAND REMOVAL — the global summary band
  (`#progressBand`: Sagas Complete / Heroic-Epic-Legendary / Rewards Banked)
  was removed entirely, along with its now-dead `renderProgress()` and
  `charSummary()` functions (their only caller) and the `.progress-band` /
  `.progress-stat` CSS. Presentation-only; the import/export contract is
  unchanged (schema still v14). Verified: full script compiles clean under
  JavaScriptCore (`jsc`, equivalent to `node --check`); no dangling refs to
  the removed functions. Browser visual pass was left to the user — the
  Claude-in-Chrome extension's site permissions blocked automated loading
  this session. Live file: `sooks-saga-scroll-07182026-1.html`. Park
  retention: kept `sooks-saga-scroll-07182026-1.html` +
  `sooks-saga-scroll-07142026-2.html` (2 most recent; 07142026-2 is also the
  cross-day anchor, so the set is 2 files); pruned
  `sooks-saga-scroll-07142026-1.html` + `sooks-saga-scroll-07122026-1.html`
  (moved to `_to_delete/`) on 2026-07-18.

  **Build 07142026.2 — panel polish: fold bar reads as a button + TIER
  label alignment (CSS + tiny JS; no schema change, still v14).** (superseded)
  (1) The "Filters & Alerts" roll-up header was bare Cinzel text and
  read as a static heading, not a control. It now carries the shared
  control chrome (parchment fill, hairline border, `--radius-ctl` +
  `--shadow-ctl`) with the tier buttons' hover language (gold border,
  `--shadow-lift`, -1px lift), a round wax-seal caret COIN
  (`.ctl-caret`, wax-red-dark, brightens on hover), and an
  always-visible italic hint stating the action ("— click to fold" /
  "— click to open", `#ctlFoldHint` set by `_applyControlsFold`; the
  hint was previously visible only while folded). (2) The TIER label
  sat flex-start while its button grid centered, floating ~50px left of
  the first button; `.ctl-row-tiers .tier-segment.bar` is now
  `justify-content: flex-start`, so the first button sits flush under
  the label — the same label-over-control relationship Group by and
  Search have (measured 2px delta = the standard label margin).
  Verified: `node --check` pass + real Chromium: label x vs first
  button x within 2px; hint text flips per state; fold/unfold +
  persistence intact; zero console errors. Live file:
  `sooks-saga-scroll-07142026-2.html`. Park retention: kept
  `sooks-saga-scroll-07142026-2.html` +
  `sooks-saga-scroll-07142026-1.html` (2 most recent) + cross-day
  anchor `sooks-saga-scroll-07122026-1.html` (3 files); pruned
  `sooks-saga-scroll-07112026-18.html` (moved to `_to_delete/`) on
  2026-07-14.

  **Build 07142026.1 — Quest Autocomplete + filter-panel roll-up
  (schema v13 → v14, additive).** (1) QUEST AUTOCOMPLETE — when the
  active character's live location (`location_id` from the 15s DDO
  Audit poll) resolves to a QUEST INSTANCE via `getQuestAreaMap()` (the
  same `area_id` → quest table the guild roster's "quest they're
  running" line uses), the scroll marks that quest complete
  automatically. Per-character setting — additive
  `characterAutoComplete` map (`{ "<char>": { on, mode } }`,
  SCHEMA_VERSION 14) — surfaced as a **Quest Autocomplete** row under
  Audio Alerts in the filter panel: an **Enabled** checkbox plus a
  **✓ Non-Reaper / ☠ Reaper** mode pair (radios dim while off).
  Defaults OFF + Non-Reaper, so nothing changes until the user opts in;
  entries follow rename/delete and round-trip via the whole-state
  Export. Non-Reaper applies the green Quest Completed seal only
  (`sagaDone` cascade through same-tier `SHARED_QUESTS` + auto-bank —
  the left checkbox's exact rules, replicated standalone in
  `_acApply()` because `wireCard`'s closures aren't reachable from the
  poll); Reaper ALSO seals First-Time Reaper (`quests[q]="reaper"`,
  the ☠ REAPER ☠ button's cascade — saga-local Skips on siblings
  respected, `NO_REAPER` solo-only quests fall back to Non-Reaper).
  Fires on ENTRY, once per stay (`_acSeen` de-dups; entry clears when
  the character is next seen outside a quest instance); tier
  disambiguated via `_sagaForQuest` with the persisted character level
  (a lvl-32 char in a shared-name quest marks the Legendary container);
  a bottom-right toast (`.ac-toast`) announces every auto-mark;
  `_acLog` keeps a debug trace; public areas can't trigger (they don't
  map to a quest `area_id`). All best-effort try/caught — offline
  behavior unchanged. (2) PANEL ROLL-UP — the whole filter/alert card
  (`.controls`) folds into a slim **"Filters & Alerts"** drawer via a
  new keyboard-reachable header bar (`#ctlFold`); state persists as
  `controlsFolded` in the META key (outside `state`, same convention as
  `guildFolded`/`raidFolded`), default open. Verified: `node --check`
  syntax pass + real Chromium (offline-forced) end-to-end: fold
  toggles/persists/unfolds; enable + mode writes land per-character;
  reaper-mode entry seals FTR + green seal + toast + log; same-stay
  de-dup holds; leaving clears the stay; non-reaper touches only the
  green seal; NO_REAPER falls back; disabled never fires; rename
  migration follows; build stamp + schema v14 + zero console errors.
  Live file: `sooks-saga-scroll-07142026-1.html`. Park retention: kept
  `sooks-saga-scroll-07142026-1.html` +
  `sooks-saga-scroll-07122026-1.html` (2 most recent; 07122026-1 is
  also the previous day's last build = the cross-day anchor) +
  `sooks-saga-scroll-07112026-18.html` (third slot, next most recent);
  pruned `sooks-saga-scroll-07112026-17.html` (moved to `_to_delete/`)
  on 2026-07-14.

  **Build 07122026.1 — four user-reported fixes: live-refresh gate,
  per-character Audio Alerts (schema v12 → v13), responsive pass, forum
  link.** (1) LIVE-REFRESH GATE — the 15s `liveRefreshTick()` returns
  early while the Add Character / Manage panel is open
  (`charManageMode`): the tick's `renderChars()` was rebuilding the
  panel DOM every 15s and wiping half-typed names and pending theme /
  server picks; an instant catch-up tick fires when the panel closes
  (Done, or a successful Add). (2) PER-CHARACTER AUDIO ALERTS —
  additive `characterAudio` map (`{ "<char>": {low,mid,high,raid} }`,
  SCHEMA_VERSION 13): checkbox writes go to the ACTIVE character (entry
  seeded from the resolved prefs on first divergence so the other three
  toggles keep their inherited values); read resolution falls back
  per-field to the legacy global META value (`audioCues`, kept as the
  seed), then the defaults (High Reaper + Raids ON, Low + Mid OFF) — so
  existing saves behave identically until a checkbox is touched;
  entries follow rename/delete; round-trips via the whole-state Export;
  `syncAudioChecks()` re-syncs the checkboxes on every character
  switch. (3) RESPONSIVE PASS — ≤1179px the two FIXED corner clusters
  drop into the document flow as centered wrapping card rows
  (align-items: flex-start, nothing can overlap the title/controls);
  1180–1620px the scroll cedes side gutters (`max-width: calc(100vw -
  452px)`) so the fixed cards never sit on the parchment; the
  fixed-width control standards (`--ctl-w`, the 2× search field) became
  caps via `max-width: 100%`; ≤480px phone tightening (body/scroll
  padding, title 2rem, stat band wrap, full-width cards). Browser ZOOM
  shrinks the CSS viewport that media queries measure, so the same
  queries fix the reported zoom collisions. (4) FORUM LINK — the
  project's DDO Forums thread linked directly under the creator credit
  (`.credit-forum`, `.credit-data` styling, new tab + noopener).
  Verified: jsdom 42 checks ALL PASS (typed text + same input node
  survive a tick with the panel open; Done/Add close paths fire the
  catch-up without throwing; v12 save re-stamps to v13 with all fields
  intact; per-field legacy fallback; divergence seeding; rename/delete
  follow; ensureShape preserves imported `characterAudio`; forum link
  placement; footer stamp) + real Chromium 28 checks ALL PASS (7 widths
  360–1920px: ZERO corner-card/content bounding-box intersections, zero
  horizontal overflow, correct fixed/static regime per width; typing +
  theme pick survive a real tick end-to-end; per-char audio writes +
  checkbox re-sync on medallion switch; schema v13 stamped) +
  screenshots at all 7 widths. Live file:
  `sooks-saga-scroll-07122026-1.html`. Park retention: kept
  `sooks-saga-scroll-07122026-1.html` +
  `sooks-saga-scroll-07112026-18.html` (2 most recent; .18 is also
  yesterday's last build = the cross-day anchor) +
  `sooks-saga-scroll-07112026-17.html` (third slot, next most recent);
  pruned `sooks-saga-scroll-07052026-12.html` on 2026-07-12.

  **Build 07112026.18 — Suggested-default migration widened (JS only,
  no schema change, still v12).** User report: with a save carrying a
  legacy `band:<tier>` tierFilter (set by the old chip's "Show <tier>"
  button), .17 lit the band tiers but not the gold Suggested button.
  Fix: `_applySuggestedDefaultMigration()` (now a named, testable
  function) migrates `band:*` values to "suggested" UNCONDITIONALLY —
  no current UI can set a band value, so a stored one is always legacy
  suggested-behavior; this also rescues saves that already booted .17
  and had the META flag set with a band value stored. "all" still
  migrates only once (flag-gated); explicit single-tier picks are
  respected. Verified: jsdom 73 checks ALL PASS (including the exact
  user case: band value + flag already set → Suggested applied; and
  explicit "Heroic" untouched) + real Chromium reproducing the user
  case end-to-end (tierFilter "suggested", Suggested + band buttons
  active). Live file: `sooks-saga-scroll-07112026-18.html`. Park
  retention: kept `sooks-saga-scroll-07112026-18.html` +
  `sooks-saga-scroll-07112026-17.html` + cross-day anchor
  `sooks-saga-scroll-07052026-12.html` (3 files); pruned
  `sooks-saga-scroll-07112026-16.html` on 2026-07-11.

  **Build 07112026.17 — "Suggested" tier-filter button (markup + CSS +
  JS, no schema change, still v12 — "suggested" is an ADDITIVE
  tierFilter VALUE; the field round-trips as before).** The
  level-suggestion CHIP row is gone; its job moved into the tier filter
  as a gold-accented **Suggested** button next to All — and it is the
  DEFAULT. Resolution is dynamic at render time
  (`suggestedBandFilter`/`resolveTierFilter`): the active character's
  persisted level maps 1–19 → Heroic, 20–29 → Epic, 30+ → Legendary and
  applies the band (saga + non-saga sibling via the existing
  `TIER_BANDS`); no known level → behaves like "all". When selected,
  the Suggested button AND its resolved band tiers all light, so the
  pick is visible. Defaults: fresh-install default and the
  invalid-value fallback are now "suggested"; **the `VALID_TIER_FILTERS`
  whitelist gained "suggested"** (found the hard way: the validator was
  stomping the new value back to "all" on load — caught by real-Chromium
  QA disagreeing with jsdom, then fixed and verified in BOTH engines);
  a ONE-TIME migration (META flag `suggestedDefaultApplied`) moves
  existing saves off the old default "all", after which any explicit
  choice persists — including All. The old chip's fold-all side effect
  was dropped (filters filter; they don't fold). Search still overrides
  and dims the row. Verified: jsdom 71 checks ALL PASS (default state,
  band lighting, list filtered to the band, dynamic re-resolution at
  lvl 12, no-level → all, All persists / Suggested returns, migration
  flag, chip gone) + real-Chromium confirmation of the same, +
  screenshots at 3 widths (8 uniform buttons, no overflow). Live file:
  `sooks-saga-scroll-07112026-17.html`. Park retention: kept
  `sooks-saga-scroll-07112026-17.html` + `sooks-saga-scroll-07112026-16.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-15.html` on 2026-07-11.

  **Build 07112026.16 — DESIGN COHESION pass (CSS only, no schema
  change, still v12).** An audit found real drift: 14 distinct corner
  radii, 30 font sizes, 58 shadow variants, 9 spellings of the font
  stacks. A token system now lives in `:root` and every surface
  references it. **Radii** — `--radius-card` 10px (corner cards,
  controls panel, saga cards, progress band, bonus banner, story/details
  panels), `--radius-ctl` 8px (every button, input, textarea, chip),
  `--radius-tag` 6px (plates, tags, header caps, the parchment edge),
  `--radius-pill` 999px (capsule family: char chips, FTR/astral/done
  badges, completion controls, level pill, progress bar — the browser
  clamps to half-height, so every pill is a true capsule). Intentional
  geometry kept: circles, the asymmetric `.quest-live`/`.qd-puzzle`
  left-rule shapes, the ☠ tag's tuned plate. **Frames** — one hairline
  (`--frame-line`) plus a 3-step shadow scale
  (`--shadow-card`/`-ctl`/`-lift`) on structural surfaces; glow/pulse
  shadows are signals and untouched. **Fonts** — three roles:
  `--font-display` (Cinzel), `--font-body` (EB Garamond stack),
  `--font-flavor` (IM Fell); 'Uncial Antiqua' title and 'IM Fell DW
  Pica' stay as intentional accents; sizes snapped to 0.64 / 0.72 /
  0.8 / 0.85 / 0.9 / 0.95 / 1+ rem (30 → 7 sizes below 1rem). Verified
  headless (jsdom, 62 checks, ALL PASS — includes token-presence
  assertions: ≥40 radius refs, ≥80 font refs, zero stray font stacks)
  and Chromium screenshots at 3 widths (no overflow, tier buttons
  uniform, all cards visually siblings). Live file:
  `sooks-saga-scroll-07112026-16.html`. Park retention: kept
  `sooks-saga-scroll-07112026-16.html` + `sooks-saga-scroll-07112026-15.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-14.html` on 2026-07-11.

  **Build 07112026.15 — BONUS DAYS banner, GIST edition (markup + CSS +
  JS, no schema change, still v12).** The .14 verdict stands for DDO's
  own properties (ddo.com / official Discord / X all refuse anonymous
  browser reads), so the data now lives in a source the USER controls
  that explicitly allows browser reads: a GitHub gist —
  `gist.githubusercontent.com` serves raw files CORS-open. **Verified
  live before shipping**: a real cross-origin fetch of the exact gist
  URL returned the JSON readable. `refreshBonusBanner()` fetches
  `DDO_BONUS_GIST_URL`
  (`gist.githubusercontent.com/eddiefiggie/0cf713ba2124e6bf1afb0b0e99ab3702/raw/ddo-bonus.json`,
  the unversioned raw form so gist edits always show) on EVERY app load
  (`cache: no-store`), validates `{ headline, expires "YYYY-MM-DD",
  details?, updated? }`, and shows the gold band under the subtitle
  while `expires` (end of that local day) is not past — fetch failure /
  malformed / lapsed → NO banner, console note says why; all fields
  HTML-escaped. **Update ritual:** when a new Bonus Days drops, edit the
  gist's two lines (gist.github.com → Edit → Update), or ask Claude to
  "update the bonus gist"; the HTML file never changes. Gist ships
  seeded with the real current event (+5% VIP XP & Double Mysterious
  Remnants Weekend, thru 2026-07-20, per the official @DDOUnlimited
  announcement). Verified headless (jsdom, 60 checks, ALL PASS): boot
  fetch + full render (headline/details/thru Jul 20/as of date), exact
  gist URL, expiry-kill, unreachable → hidden without errors, HTML
  injection escaped, recovery on valid data, old wiki plumbing still
  absent, and every surviving Build .1–.14 behavior. Chromium
  screenshot: banner rendered with the live event. Live file:
  `sooks-saga-scroll-07112026-15.html`. Park retention: kept
  `sooks-saga-scroll-07112026-15.html` + `sooks-saga-scroll-07112026-14.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-13.html` on 2026-07-11.

  **Build 07112026.14 — DAILY BONUS banner REMOVED after live QA + live
  population source (markup + CSS + JS, no schema change, still v12).**
  LIVE QA — run in a real browser tab against the real services (via
  Chrome MCP) — settled the banner question with evidence: ddowiki's
  `api.php` AND `rest.php` are firewalled behind the host's load
  balancer (HTTP 202, empty body, even SAME-ORIGIN — so JSONP, which
  rides api.php, is equally dead), and the regular page HTML, though it
  contains the coupon table server-rendered, sends NO
  `Access-Control-Allow-Origin` header — unreadable by a local file.
  DDO Audit's full OpenAPI surface (66 routes via `/docs/openapi.json`)
  was enumerated: no bonus/coupon data there either (`/v1/service/news`
  and `/v1/service/page_messages` are empty notice feeds). Conclusion:
  **no automatable Bonus Days source is reachable from a local HTML
  file**; per the project rule (only self-updating data), the banner and
  ALL its plumbing were removed (markup, CSS, `BONUS_WIKI_URLS`,
  `_wikiJsonp`, `_extractActiveCoupon`, `_couponActive`,
  `refreshDailyBonus`, `renderBonusBanner`, boot + visibilitychange
  hooks; stale META `dailyBonus` values ignored). SILVER LINING from the
  same live probe: `/v1/population/timeseries/day`'s newest datapoint
  carries LIVE per-server `character_count` (verified real values, e.g.
  thrane 633) — `fetchServerPopulation` now uses it as PRIMARY (cheap,
  current), who-list count as fallback; `/game/server-info` dropped as a
  source (live data shows status/queue only, no counts). Also fixed a
  real bug the new test caught: a missing server entry coerced
  `Number(null)` → fake "0 online" that blocked the fallback (guard is
  now `!= null`). Verified headless (jsdom, 54 checks, ALL PASS):
  banner + all its JS confirmed absent, pop bar reads the NEWEST
  datapoint (1,234 not the older 900), fallback fires when the server is
  missing from the timeseries, and every surviving Build .1–.13 behavior
  passes. Live file: `sooks-saga-scroll-07112026-14.html`. Park
  retention: kept `sooks-saga-scroll-07112026-14.html` +
  `sooks-saga-scroll-07112026-13.html` + cross-day anchor
  `sooks-saga-scroll-07052026-12.html` (3 files); pruned
  `sooks-saga-scroll-07112026-12.html` and the diagnostic
  `ddowiki-connectivity-test.html` on 2026-07-11.

  **Build 07112026.13 — the daily-check stamp no longer locks in an
  EMPTY result (JS only, no schema change, still v12).** The once-a-day
  short-circuit in `refreshDailyBonus` now also requires an ACTIVE
  coupon on record (`_couponActive`) — a check that found nothing
  retries on every load instead of stamping "checked today — nothing"
  and silently blocking all later attempts until midnight. That trap is
  exactly why the .12 parser/transport fixes never ran for the user: an
  older build had already stamped the day empty, and with the banner
  hidden there was no ↻ to force past it. Successful results keep the
  once-a-day cadence. Verified headless (jsdom, 70 checks, ALL PASS)
  including the new case: same-day stamp with no coupon → next load
  re-fetches and recovers the banner. Live file:
  `sooks-saga-scroll-07112026-13.html`. Park retention: kept
  `sooks-saga-scroll-07112026-13.html` + `sooks-saga-scroll-07112026-12.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-11.html` on 2026-07-11.
  **Build 07112026.12 — bonus banner robustness: parser + transport (JS
  only, no schema change, still v12).** User reported the banner not
  appearing; two likely causes fixed blind (the authoring sandbox has no
  egress). (1) **Parser:** a Coupons-table row with a SINGLE date is an
  expiry-only row ("Expires July 16, 2026") — previously misread as
  start=expiry ("not yet begun") and skipped, which would hide the
  banner even with perfect connectivity. Single-date rows now count as
  already-started; full-range rows keep the not-yet-begun skip; among
  active rows the LATEST EXPIRY wins. (2) **Transport:** if the normal
  CORS fetch to `api.php` fails, a JSONP fallback (`_wikiJsonp`:
  script-tag callback — MediaWiki's classic pre-CORS mechanism, immune
  to CORS; 12s timeout, self-cleaning) is tried before giving up.
  Diagnostics name the transport ("reached via fetch/jsonp") and a
  hidden banner now `console.warn`s its reason (previously invisible —
  no tooltip to hover on a hidden element). Verified headless (jsdom,
  69 checks, ALL PASS): expiry-only rows parse (boot table uses single
  dates), JSONP rescue path (fetch down → jsonp payload → banner
  returns), both-transports-down keeps an unexpired coupon and never
  resurrects an expired one, plus the full .11 lifecycle and every
  Build .1–.11 behavior. Live file:
  `sooks-saga-scroll-07112026-12.html`. Park retention: kept
  `sooks-saga-scroll-07112026-12.html` + `sooks-saga-scroll-07112026-11.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-10.html` on 2026-07-11.

  **Build 07112026.11 — bonus banner EXPIRY-KILL (JS only, no schema
  change, still v12).** Per user design: the banner dies the moment the
  Bonus Days event lapses, and the daily check then watches for the next
  one. `_extractActiveCoupon` now returns `{ text, expiry }` (expiry =
  end of the coupon's final day; stored as `couponExpiry` in the META
  `dailyBonus` object); the new `_couponActive()` gates both the
  renderer (expired → banner hides entirely) and the daily check's
  carry-forward (an expired coupon is NOT kept as "last known" when the
  wiki is unreachable or lagging — it dies on schedule; a legacy coupon
  without an expiry is treated as unknown and replaced on the next
  successful check). Diagnostics distinguish "previous coupon expired —
  awaiting the next event" from unreachable/parse states. ALSO
  investigated and ruled out: probing `ddo.com/news/ddo-bonus-<MMDDYY>`
  URLs from the app — those pages are empty JS shells (article text
  loads via a private script API) AND ddo.com sends no CORS headers, so
  a local file cannot read or even existence-check them; the slugs are
  per-cycle, not daily, besides. Verified headless (jsdom, 68 checks,
  ALL PASS): couponExpiry persisted, expired coupon kills the banner,
  wiki-down daily check does not resurrect it (diag notes the expiry),
  a later check with the next event restores the banner, and every
  Build .1–.10 behavior intact. Live file:
  `sooks-saga-scroll-07112026-11.html`. Park retention: kept
  `sooks-saga-scroll-07112026-11.html` + `sooks-saga-scroll-07112026-10.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-9.html` on 2026-07-11.

  **Build 07112026.10 — bonus banner is AUTOMATED-ONLY (markup + CSS +
  JS, no schema change, still v12).** Per user direction ("only keep the
  data that can be automated"), the Build .9 manual Discord-pasted bonus
  line, its ✎ editor (`_bonusEdit`), and the `.bb-text`/`.bb-unset`/
  `.bb-input` CSS were REMOVED. The banner now shows only the
  wiki-sourced Bonus Days coupon ("⚑ Bonus Days · 🎟 <code> · freshness
  chip · ↻"), still auto-checked once per calendar day, and **hides
  entirely** when no automated data is available (old
  `manualText`/`manualSetAt` META values are ignored). Candidate for a
  future build: the wiki's [Bonus Days](https://ddowiki.com/page/Bonus_Days)
  page may track the weekly in-game bonus — unverifiable from the
  authoring sandbox (robots blocks the fetcher's API route; the rendered
  page came back empty) — if it does, the bonus sentence can be
  automated with the same api.php mechanism as the coupon. Verified
  headless (jsdom, 64 checks, ALL PASS): no manual hint/editor remains,
  coupon parse + once-a-day guard + ↻ + day rollover + unreachable
  fallback intact, banner hides at `coupon: null`, all Build .1–.9
  behaviors pass. Chromium screenshot: single-line automated banner.
  Live file: `sooks-saga-scroll-07112026-10.html`. Park retention: kept
  `sooks-saga-scroll-07112026-10.html` + `sooks-saga-scroll-07112026-9.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-8.html` on 2026-07-11.

  **Build 07112026.9 — DAILY BONUS banner (markup + CSS + JS, no schema
  change, still v12; state in the META key's `dailyBonus` object).**
  A slim gold band under the subtitle (`#bonusBanner`) with two halves.
  (1) **The in-game bonus line is USER-SET** — Discord exposes no
  anonymous API and a local file cannot read the official DDO server, so
  the ✎ button swaps in an input (Enter/blur saves, Esc cancels; text
  persists with a "(set Jul 11)" chip). (2) **The Bonus Days coupon is
  AUTOMATIC** — fetched from the DDO Wiki's Coupons page via the
  CORS-open MediaWiki `api.php` (`origin=*`), parsed by
  `_extractActiveCoupon` (table-row scan: ALL-CAPS code + a date range
  covering today, newest start wins; cell texts joined with spaces —
  `textContent` alone glues cells; any parse failure keeps the last
  known coupon). Checks run at most **once per calendar day**
  (`checkedDay` guard) at load and on visibilitychange (day rollover for
  long-lived tabs); ↻ forces a re-check; a "checked today / last
  checked <date>" chip shows freshness and the tooltip self-diagnoses
  (reached/parsed/HTTP/unreachable). CAVEAT: the authoring sandbox has
  no network egress, so the live wiki CORS path wasn't exercised
  pre-park — it mirrors the working DDO Audit fetch pattern exactly and
  degrades silently; the tooltip reports what actually happened on
  first real load. Verified headless (jsdom, 65 checks, ALL PASS):
  0 runtime errors, stamp `07112026.9`; active coupon parsed (expired
  row skipped), once-a-day guard (1 fetch at boot, same-day no-op,
  ↻ re-fetches, day-rollover re-fetches), ✎ edit persists + renders
  with set-date, state stays out of `state` (schema untouched), wiki-
  unreachable keeps the last coupon + records a diagnostic, and every
  Build .1–.8 behavior intact. Chromium screenshot: banner shows bonus
  line + coupon + freshness chip + ✎/↻. Live file:
  `sooks-saga-scroll-07112026-9.html`. Park retention: kept
  `sooks-saga-scroll-07112026-9.html` + `sooks-saga-scroll-07112026-8.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-7.html` on 2026-07-11.

  **Build 07112026.8 — footer credit legibility (CSS only, no schema
  change, still v12).** The `.credit-data` line inherited the footer's
  italic IM Fell + faded ink and read like hard-to-parse script. It now
  opts out: upright (`font-style: normal`) EB Garamond, 0.78 → 0.92rem,
  full-strength `--ink-secondary`, and the DDO Audit / Clemeit links are
  bold wax-red with a solid (was dotted) underline. No markup/JS change.
  Verified headless (jsdom, 53 checks, ALL PASS — the .7 suite + the new
  CSS assertions) and by Chromium footer screenshot. Live file:
  `sooks-saga-scroll-07112026-8.html`. Park retention: kept
  `sooks-saga-scroll-07112026-8.html` + `sooks-saga-scroll-07112026-7.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-6.html` on 2026-07-11.

  **Build 07112026.7 — FTR spotlight + DDO Audit credit + server bar +
  2× refresh (markup + CSS + JS, no schema change, still v12).**
  (1) **FIRST TIME REAPER spotlight** in the reaper cards:
  `_ftrNeeded(quest, minL, maxL)` is true when the quest maps to one of
  the scroll's containers (tier-disambiguated via `_sagaForQuest`), is
  reaper-eligible (not in `NO_REAPER`), and the active character's
  per-quest state isn't `"reaper"` yet. Such rows are DRAMATIC —
  `tr.ftr-need` gold wash + 4px gold left rule + pulsing inner glow
  (`ftrRowPulse`, reduced-motion gated) + a bold "✦ First Time Reaper ✦"
  banner (`.rd-ftr`) under the quest name — and they **sort to the TOP**
  of their card ahead of skull count. Claiming the reaper seal clears
  the spotlight on the next repaint. (2) **DDO Audit credit** in the
  footer (`.credit-data`): courtesy line linking
  [ddoaudit.com](https://www.ddoaudit.com) and Clemeit (the project's
  author). (3) **Server bar** (`#serverPop`, `.pop-bar`) above the guild
  card: "⚘ <Server> · N online". `fetchServerPopulation()` tries the
  small `/game/server-info` payload for a per-server character count
  (several field spellings accepted) and falls back to counting the
  `/characters/<server>` who-list; own 30s cache (`_popCache`/`POP_TTL`)
  so the heavy fallback fires at most twice a minute; count omitted
  while unknown; stale-guarded. (4) **Refresh doubled**:
  `LIVE_REFRESH_MS` 30000 → 15000 (card tooltips updated) — the LFM
  window and live caches were already tick-driven, so all live data now
  pulls every 15s. Verified headless (jsdom, 52 checks, ALL PASS):
  0 runtime errors, self-check unchanged, stamp `07112026.7`; FTR
  spotlight on a container quest (banner + row class), FTR sort-to-top
  (R8 FTR above R9 non-FTR), spotlight clears when the seal is claimed,
  `NO_REAPER` never spotlights, footer credit links exact, server bar
  reads 1,234 from server-info and 77 from the who-list fallback,
  15s constant + tooltips, and every Build .1–.6 behavior intact.
  Chromium screenshots: glowing FTR rows sorted first, server bar above
  the guild card, credit line in the footer. Live file:
  `sooks-saga-scroll-07112026-7.html`. Park retention: kept
  `sooks-saga-scroll-07112026-7.html` + `sooks-saga-scroll-07112026-6.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-5.html` on 2026-07-11.

  **Build 07112026.6 — wiki links + Low/Mid reaper cards + audio-alert
  checkboxes (markup + CSS + JS, no schema change, still v12).**
  (1) **Quest/raid names in every LFM card are DDO Wiki hyperlinks** —
  `_wikiHref()` builds `https://ddowiki.com/page/<Name_with_underscores>`
  (standard MediaWiki naming; links only, no wiki content fabricated),
  `target="_blank" rel="noopener"`, dotted-underline ink styling
  (`.rd-name a`). (2) **The reaper stack is three cards** — Low ☠ R1–R3
  (`#lowCorner`), Mid ☠ R4–R7 (`#midCorner`), High ☠ R8–R10
  (`#highCorner`), top-to-bottom above the Raids card — all rendered by
  the shared `renderReaperCorner()` (`REAPER_CARDS` config,
  `renderReaperCorners()` fan-out) over ONE `_reaperLfms` list:
  `refreshLfmIndex` now collects every declared-skull (R1+) non-raid
  quest group, one signature `_reaperSig` repaints all three. Rows keep
  the .5 shape (linked name / ❧ saga-or-pack / range · difficulty ·
  ☠ R<n> / leader · fill/6; neutral/.mid/.high tint per band).
  **Every LFM card (three reaper tiers + Raids) now DEFAULTS TO
  COLLAPSED** — `_foldDefault()` reads an unset META value as folded;
  explicit unfolds persist (`lowFolded`/`midFolded`/`highFolded`/
  `raidFolded`); headers keep showing live counts while collapsed.
  (3) **AUDIO ALERTS checkbox row** in the filter panel under the tier
  filter — Low Reaper / Mid Reaper / High Reaper / Raids (`#audioLow`/
  `#audioMid`/`#audioHigh`/`#audioRaid`, `bindAudioChecks()`), each
  gating its cue (knell for reaper tiers, horn for raids) at PLAY time.
  Per-band seen-sets (`_seenLowLfms`/`_seenMidLfms`/`_seenHighLfms`)
  stay maintained while a cue is off, so re-enabling never rings for
  groups already up. Prefs persist in the META key (`audioCues`);
  defaults preserve prior behavior — High + Raids ON, Low + Mid OFF.
  Verified headless (jsdom, 38 checks, ALL PASS): 0 runtime errors,
  self-check unchanged (✓ 62 containers / 517 quests / 100% detailed),
  stamp `07112026.6`; card order Low→Mid→High→Raids, all four default-
  collapsed with live counts in the headers, unfold persistence, wiki
  hrefs exact (`Temple_of_the_Deathwyrm`, `The_Final_Enemy`,
  `Killing_Time`), R2 row in the low card (neutral tint, saga line,
  Pip · 2/6), checkbox defaults + META persistence, Mid-OFF silent +
  Mid-ON rings, Raids-OFF silent while the card still updates, and every
  Build .1–.5 behavior intact. Chromium screenshots: collapsed stack
  (compact headers) + unfolded stack + audio row under the tier filter;
  zero horizontal overflow. Live file:
  `sooks-saga-scroll-07112026-6.html`. Park retention: kept
  `sooks-saga-scroll-07112026-6.html` + `sooks-saga-scroll-07112026-5.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-4.html` on 2026-07-11.

  **Build 07112026.5 — saga / story-pack line in the High Reaper card
  (CSS + JS, no schema change, still v12).** Each High Reaper row now
  shows, UNDER the quest name, which saga or story pack the quest belongs
  to (`.rd-saga`: quiet italic "❧ <name>", slot reserved when unknown);
  row order is name → saga/pack → range · difficulty · ☠ R<n> → leader ·
  fill/6. Source order: the scroll's own containers first —
  `_sagaForQuest(questName, minL, maxL)` scans `SAGAS` by normalized
  quest name and, when the same quest name exists in multiple tiers
  (Heroic vs Legendary Saltmarsh), picks the container whose tier band
  overlaps the group's posted level range (same `LFM_TIER_BANDS` mapping
  `lfmForTier` uses); true sagas show their saga `name`, non-saga
  containers show their `pack`. Fallback for quests outside every
  container: the quest's `required_adventure_pack` from DDO Audit — the
  quests ref cache rows now carry `pack` (shape guard refetches older
  caches once, same pattern as `area`/`raid`), `_setQuestMaps` builds
  `_questPackMap`, and each high entry carries its pack (folded into
  `_highSig`). Verified headless (jsdom, 65 checks, ALL PASS): 0 runtime
  errors, self-check unchanged (✓ 62 containers / 517 quests / 100%
  detailed), stamp `07112026.5`; row order asserted, out-of-container
  quest falls back to "❧ The Soul Splitter", in-container quest resolves
  to "❧ Sinister Secret of Saltmarsh" with `_sagaForQuest("The Final
  Enemy", 30, 34)` → `l-saltmarsh` and `(…, 1, 4)` → `h-saltmarsh`
  (tier disambiguation), skulls-first sort intact, and every Build .1–.4
  raids/high/audio/LFM/controls behavior still passes. Chromium
  screenshot confirms the card reads name / ❧ saga-or-pack / range ·
  difficulty · skulls / leader · fill. Live file:
  `sooks-saga-scroll-07112026-5.html`. Park retention: kept
  `sooks-saga-scroll-07112026-5.html` + `sooks-saga-scroll-07112026-4.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-3.html` on 2026-07-11.

  **Build 07112026.4 — guild card moved upper-left + HIGH REAPER card
  (markup + CSS + JS, no schema change, still v12).** (1) The **Guild card
  now floats in its own cluster in the UPPER-LEFT corner** of the screen
  (`.data-corner.left`: `right:auto; left:14px`, mirrored to 8px in the
  ≤640px media rule) — same card, same fold, only the anchor moved. The
  right cluster now stacks **Characters → High Reaper → Raids**. (2) New
  **HIGH REAPER card** (`#highCorner`, reuses `.raid-panel` card
  mechanics) directly ABOVE the raids card: live **☠ R8–R10 QUEST groups**
  (raid quests excluded — the raids card covers those, matching the
  audio-cue exclusion) that are level-relevant to the active character —
  exactly the set the reaper knell rings for. `refreshLfmIndex` collects
  `highs` alongside `raids` on the same fetch/30s tick (`_highLfms`, own
  signature guard `_highSig`), and the knell's key set now derives from
  that same list (one source of truth, identical semantics). Same 3-line
  rows as the raids card — `.rd-name` / `.rd-meta` (range · difficulty ·
  ☠ R<n>, always declared here) / `.rd-live` (leader · fill) — but the
  fraction is over the **6-player quest-group cap** ("2/6"), and the live
  line is always `.high`-tinted. Folds like its siblings (`highFolded` in
  the META key); renderAll, the 30s tick, and `setCharacterLevel` repaint
  it; hidden when no server is set. Verified headless (jsdom, 59 checks,
  ALL PASS): 0 runtime errors, self-check unchanged (✓ 62 containers /
  517 quests / 100% detailed), stamp `07112026.4`; guild card verified
  inside `.data-corner.left`, `#highCorner` verified directly above
  `#raidCorner` in the right cluster; empty state at boot, R9 group
  renders 3 rows (name / `lvl 30–32 · Reaper · ☠ R9` / `Grimjaw · 2/6`),
  R8 raid stays OUT of the high card, fold + META persistence, no-server
  hide, and every Build .1–.3 raids/audio/LFM/controls behavior intact.
  Chromium screenshots (1500/1000/760px): guild card upper-left, High
  Reaper above Raids on the right, zero horizontal overflow, tier buttons
  still uniform. Live file: `sooks-saga-scroll-07112026-4.html`. Park
  retention: kept `sooks-saga-scroll-07112026-4.html` +
  `sooks-saga-scroll-07112026-3.html` + cross-day anchor
  `sooks-saga-scroll-07052026-12.html` (3 files); pruned
  `sooks-saga-scroll-07112026-2.html` on 2026-07-11.

  **Build 07112026.3 — three UI cleanups (markup + CSS + JS, no schema
  change, still v12).** (1) **Raid rows are three stacked lines** —
  `.rd-name` (raid name), `.rd-meta` (posted level range · difficulty ·
  `☠ R<n>` when declared), `.rd-live` (leader · fill/12, still tinted
  `.mid`/`.high` by skulls) — replacing the cramped two-line blend; the
  indented lines reserve their slot so rows read uniform. (2) **The
  per-tier progress-bar band is GONE**: `#tierRollup` markup,
  `renderProgress()`'s rollup branch, the `tierRollup()` helper, all
  `.tier-rollup*`/`.rollup-*` CSS, and the narrow-screen rules — removed
  per user request (a series of five bars under the counters, too many).
  The Sagas Complete / Heroic-Epic-Legendary / Rewards Banked counters
  band stays; per-saga bars in container headers stay (LFM tints ride
  them). (3) **The filter/search controls are ONE tidy panel**:
  `.controls` is now a bordered parchment card (max-width 960px, 14px
  radius, soft shadow, subtle gradient) holding two centered rows —
  row 1 = Group-by · Search (two units wide) · Expand-all, row 2 = the
  tier filter. Every button shares ONE width standard `--ctl-w: 172px`,
  keyed to the widest label ("Non-Saga Legendary"); the tier row is a
  centered flex-wrap of fixed-width buttons, so wrapped lines re-center
  and the block stays symmetric at any viewport (7 across wide → 3+3+1
  narrow); the old nowrap horizontal-scroll strip and the `seg-all`
  margin offset are gone — the page NEVER widens. Buttons modernized:
  9px radii, hairline borders, hover lift + gold border, focus-visible
  rings, wax-red active fill with a soft glow. All hooks (`#groupBy`,
  `#tierFilter`, `#questSearch`, `#searchClear`, `#searchCount`,
  `#expandAll`) unchanged. Verified two ways. Headless (jsdom, 46
  checks, ALL PASS): 0 runtime errors, self-check unchanged (✓ 62
  containers / 517 quests / 100% detailed), stamp `07112026.3`, 3-line
  raid structure in order name→meta→live, `#tierRollup`/`tierRollup()`
  gone while counters + per-saga bars remain, panel structure + tier
  clicks + search dimming + Expand-all toggle work, and every Build
  .1/.2 raids/audio/LFM behavior intact. Visually (real Chromium
  screenshots at 1500/1000/760px): zero horizontal overflow at every
  width, all 7 tier buttons exactly equal width, no clipped labels,
  wrapped rows re-center. One caveat noted while testing: a synthetic
  localStorage seed missing `groupBy` renders the Group-by select blank
  (selectedIndex −1) — pre-existing on the parked build too, and real
  saves always carry the field; not a regression. Live file:
  `sooks-saga-scroll-07112026-3.html`. Park retention: kept
  `sooks-saga-scroll-07112026-3.html` + `sooks-saga-scroll-07112026-2.html`
  + cross-day anchor `sooks-saga-scroll-07052026-12.html` (3 files);
  pruned `sooks-saga-scroll-07112026-1.html` on 2026-07-11.

  **Build 07112026.2 — left-justified card title bars + audio cues (CSS +
  JS, no schema change, still v12).** (1) `.guild-table thead th`
  `text-align` center → left (padding `5px 8px`) — the Guild AND Raids
  title bars are now left-justified (shared rule, one change). (2) Two
  synthesized WebAudio cues, zero external audio files (the scroll stays
  single-file/offline): a low tritone **knell** (G3→C#3, triangle) when a
  new level-relevant **HIGH reaper** (☠ R8–R10 declared) quest group
  appears in the LFM feed, and a rising **horn** swell (F3→C4, sawtooth)
  when a new level-relevant **raid** group appears — an R8+ raid rings
  the horn only, never both. Novelty = the (quest, leader) pair vs the
  previous pass (`_seenHighLfms` / `_seenRaidLfms` Sets rebuilt in
  `refreshLfmIndex`), so party-fill/comment churn never re-triggers; at
  most one cue per kind per pass; the first index build after load seeds
  the sets silently (no chorus on boot). Relevance shares the raids
  card's rule via new helpers `_lvlRelevant(minL, maxL, lvl)` /
  `_activeCharLevel()` (renderRaidCorner refactored onto them — one
  source of truth). Autoplay-safe: lazy `AudioContext` (`_audioCtxGet`),
  and if the browser suspends it pending a gesture, a one-time
  pointerdown/keydown listener resumes it; `_playCue` is fully
  try/caught (master gain 0.16 — a cue, not an alarm) and logs to
  `_cueLog` for debugging/tests. Verified headless (jsdom + AudioContext
  stub, 26 checks, ALL PASS): 0 runtime errors, self-check unchanged
  (✓ 62 containers / 517 quests / 100% detailed), build stamp
  `07112026.2`; boot seeds silently even with an R8 raid already up;
  fill churn (7/12 → 11/12) and raid-disappearance play nothing; a new
  relevant raid rings the horn exactly once (and NOT the knell despite
  its r8 comment); a new relevant R9 quest group rings the knell; steady
  feed, irrelevant-level raids, and new R4 groups stay silent; fold/
  unfold + META persistence and every Build 07112026.1 raids-card and
  LFM behavior intact. Live file: `sooks-saga-scroll-07112026-2.html`.
  Park retention: kept `sooks-saga-scroll-07112026-2.html` +
  `sooks-saga-scroll-07112026-1.html` + cross-day anchor
  `sooks-saga-scroll-07052026-12.html` (3 files); nothing pruned on this
  park.

  **Build 07112026.1 — RAIDS card under the guild list (markup + CSS + JS,
  no schema change, still v12).** A third collapsible corner card
  (`#raidCorner`, `.raid-panel`) below the Guild panel showing live raid
  LFMs relevant to the active character's level. Data plumbing: the quests
  ref cache (`sooksSagaScrollRef`) rows now carry `raid`
  (`group_size === "Raid"` from `/v1/quests`; a shape guard treats older
  caches without the key as stale and refetches once, same pattern as
  Build 07052026.1's `area`); `_setQuestMaps` builds `_raidQuestIds`;
  `refreshLfmIndex` splits raid LFMs into `_raidLfms` on the SAME fetch —
  raid entries also stay in the quest badge index, so nothing existing
  changed. `renderRaidCorner()` (sync, in-memory) filters by the
  character's persisted aggregate level (`state.characterLevels`): the
  group's posted range must include it; no posted range → shown
  (discoverability rule, as in `lfmForTier`); level unknown → all shown.
  Row = raid name · posted range (`gm-line1`/`gm-meta`), then a live
  sentence "☠ R8 · Xokotli · 7/12" (`.rd-live`, tinted `.mid` R4–7 /
  `.high` R8–10; leader blank → "Anonymous"; **12 = raid party cap**,
  fraction = leader + members). Sorted skulls-first, then fill. Folds
  exactly like the guild card — `raidFolded` persists in the META key
  (`sooksSagaScrollMeta`, outside `state`). Refresh: rebuilt by
  `refreshLfmIndex` under its own signature guard (`_raidSig`) on the same
  30-second `liveRefreshTick` as the reaper LFM badges; a level change via
  `setCharacterLevel` repaints the card immediately; hidden when no server
  is set (`:empty`), shows a "No level-relevant raids right now" row when
  the server is set but nothing matches. Verified headless (jsdom, 30
  checks, ALL PASS): 0 runtime errors (the lone jsdom "navigation"
  complaint reproduces identically on the parked 07052026-12 build —
  environment limitation, not a regression), self-check unchanged (✓ 62
  containers / 517 quests / 100% detailed), build stamp `07112026.1`;
  level filter (lvl-32 char: 30–34 raid shown, 10–13 raid hidden,
  no-range raid shown), leader + fraction render, fold/unfold + META
  persistence, live-tick fraction update (7/12 → 11/12), empty state,
  no-server hide, and all Build .1–.12 LFM behaviors intact (party LFMs
  still badge quests, raids still feed the quest index, tier-awareness
  intact). Live file: `sooks-saga-scroll-07112026-1.html`. Park retention:
  kept `sooks-saga-scroll-07112026-1.html` +
  `sooks-saga-scroll-07052026-12.html` (the cross-day anchor overlaps the
  recent pair — retain set is 2 files, no backfill); pruned
  `sooks-saga-scroll-07052026-11.html` and
  `sooks-saga-scroll-06302026-1.html` on 2026-07-11.

  **Build 07052026.12 — LFM matching is tier-aware (JS only, no schema
  change, still v12).** The name-normalized LFM index intentionally strips
  `(Epic)`/`(Legendary)` suffixes, so a heroic-level group used to light
  BOTH Saltmarsh containers — the user does not want heroic LFM data on the
  Legendary copy (or vice versa, or on Epic). The index now stores **each
  group's own record** (`entry.groups`: `{minL, maxL, diff, skulls, leader,
  party}`; repaint signature built from the group records), and a new
  **`lfmForTier(questName, tier)`** helper aggregates only the groups whose
  posted level range overlaps the container's tier band
  (`LFM_TIER_BANDS`: heroic 1–19, epic 20–29, legendary 30+; Non-Saga
  tiers map to the same bands). Both consumers switched over —
  `renderSagaCard`'s container scan (`lfmForTier(q, saga.tier)`) and the
  quest-row lookup (`lfmForTier(questName, tier)`) — so bars, ☠ tags, live
  lines, and row glows appear only in the tier the group is actually
  running. Edge rules: a group with no posted levels can't be classified
  and shows in all tiers (discoverability over false silence); a range
  straddling a band edge (e.g. 17–20) legitimately matches both tiers.
  Verified headless (jsdom, 74 checks, ALL PASS): 0 runtime errors,
  self-check unchanged (✓ 62 containers / 517 quests / 100% detailed),
  build stamp `07052026.12`; a lvl 1–4 R4 group lights EXACTLY ONE
  container (the Heroic Saltmarsh, verified by title) with one glowing
  row; a lvl 30–34 R8 group lights exactly the LEGENDARY Sinister Secret
  of Saltmarsh ("☠ R8 · Xokotli · 2/6"); all Build .1–.11 behaviors
  intact. Live file: `sooks-saga-scroll-07052026-12.html`. Park retention:
  kept `sooks-saga-scroll-07052026-12.html` +
  `sooks-saga-scroll-07052026-11.html` + cross-day anchor
  `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-10.html` on 2026-07-05.

  **Build 07052026.11 — ☠ tag no longer looks covered by the progress bar
  (CSS only, no schema change, still v12).** The Build .10 flush attachment
  (cell gap 0, square top corners, `border-top: none`) placed the tag
  inside the bar's box-shadow glow halo and under its rounded bottom
  corners — with the brilliant/tier glows active (`has-progress`,
  `lfm-mid`/`lfm-high` pulses spread 3–9px), the halo painted OVER the
  tag's top edge, making it look partially covered. Fix on
  `.saga-progress-cell` / `.lfm-skull-tag`: a 4px gap to clear the halo,
  full 8px corner rounding, the tag's own top border restored, and
  `position: relative` + `z-index: 1` so the tag always paints above any
  glow — it now reads as a clean label plate under the bar. Verified
  headless (jsdom, 71 checks, ALL PASS): 0 runtime errors, self-check
  unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp
  `07052026.11`, tag geometry assertions updated (130px / 8px radius /
  z-index above glow / 4px cell gap / top border present), all Build
  .1–.10 behaviors intact. Live file:
  `sooks-saga-scroll-07052026-11.html`. Park retention: kept
  `sooks-saga-scroll-07052026-11.html` +
  `sooks-saga-scroll-07052026-10.html` + cross-day anchor
  `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-9.html` on 2026-07-05.

  **Build 07052026.10 — LFM info integrated properly: structured two-line
  quest rows + attached header tag (markup + CSS, no schema change, still
  v12).** Replaces Build .9's free-wrapping chips (fixed the overlap but
  read as patchwork, per user feedback). Quest rows are now a deliberate
  two-line structure: the controls cluster stays left; a new `.quest-main`
  column holds **line 1** (`.quest-line1`: name · ↔ shared · giver · level
  chip, baseline-aligned, wraps only as a whole unit) and **line 2** — a
  dedicated `.quest-live` line rendered only when a live group exists,
  reading like a sentence: **"🔥 R5 · Henk's group · 3/6 · lvl 5–8"** (EB
  Garamond, fit-content width so it's an annotation not a banner, slim
  left rule tying it to the row's edge glow, tinted neutral wax-red /
  `.mid` orange / `.high` red; multi-group quests append "N groups"). The
  inline `.quest-lfm` chip is gone from the name line (CSS block remains,
  unused). The header **☠ tag is now an attached label** under the
  progress bar: exactly the bar's 130px width, bottom-only corner radius,
  no top border, zero gap — bar + tag read as one element. Verified
  headless (jsdom, 71 checks, ALL PASS): 0 runtime errors, self-check
  unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp
  `07052026.10`; the live line renders R4 + Henk's group + 3/6 with the
  `.mid` class, sits structurally inside `.quest-main` UNDER `.quest-line1`
  (not inline), zero `.quest-lfm` chips remain in rendered rows, the tag
  is 130px/bottom-radius/no-top-border inside `.saga-progress-cell`, and
  every Build .1–.9 behavior still passes. Live file:
  `sooks-saga-scroll-07052026-10.html`. Park retention: kept
  `sooks-saga-scroll-07052026-10.html` +
  `sooks-saga-scroll-07052026-9.html` + cross-day anchor
  `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-8.html` on 2026-07-05.

  **Build 07052026.9 — layout fix: quest-row text no longer overlaps (CSS +
  one markup wrapper; no schema change, still v12).** Two root causes, both
  fallout from Build .8's wider LFM badge. (1) `.quest` was a NON-wrapping
  flex row whose right-side chips (`.quest-giver`, `.quest-level`,
  `.quest-lfm`) are all `nowrap` + `flex-shrink: 0`; once the chips
  outgrew the row, the quest name (the only shrinkable item, `min-width:
  0`) collapsed to nothing and the rigid chips painted over the remaining
  text — the "overlapping text" the user saw. `.quest` now `flex-wrap`s
  (gap `4px 10px`) so chips drop to a tidy second line, and `.quest-lfm`
  may wrap internally (`white-space: normal`, `overflow-wrap: anywhere`,
  `max-width: 100%`) if a long leader name ever exceeds the whole row.
  (2) The header ☠ tag was appended as a SEVENTH child of the 6-column
  `.saga-header` grid, landing in an implicit 20px cell and overflowing.
  The progress bar + tag now share the last grid column inside a stacked
  flex wrapper (`.saga-progress-cell`, bar above tag, centered); the tag
  and its `.lfm-lead` wrap within the 130px column (all `nowrap` removed).
  Verified headless (jsdom, 69 checks, ALL PASS): 0 runtime errors,
  self-check unchanged (✓ 62 containers / 517 quests / 100% detailed),
  build stamp `07052026.9`; every rendered ☠ tag is structurally inside a
  `.saga-progress-cell`, the bar is in the cell, `.quest` flex-wraps, the
  badge wraps internally, no `nowrap` remains in the tag CSS, and all
  Build .1–.8 behaviors (leader/fraction text included) still pass. Live
  file: `sooks-saga-scroll-07052026-9.html`. Park retention: kept
  `sooks-saga-scroll-07052026-9.html` + `sooks-saga-scroll-07052026-8.html`
  + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-7.html` on 2026-07-05.

  **Build 07052026.8 — LFM indicators name the leader + party fill (no
  schema change, still v12).** `refreshLfmIndex` now keeps two more fields
  per index entry: `leader` — the LFM poster's name from the `leader`
  object (blank name, i.e. in-game Anonymous, renders as "Anonymous") — and
  `party` — leader + `members[]`, displayed against the 6-player
  quest-group cap. When several groups run the same quest, the displayed
  pair belongs to the group with the **highest declared skulls** (first
  wins ties) so the name always matches the tier on the tag; both fields
  fold into the repaint signature so the 30s tick updates them live (a
  member joining flips 3/6 → 4/6). Display: the header tag now reads
  **"☠ R8 · Henk · 3/6"** — the leader/party half is plain readable EB
  Garamond (`.lfm-lead`, no small-caps letterspacing, inherits the chip
  color); the per-quest badge reads **"🔥 1 LFM · R5 · Henk 3/6"**; both
  tooltips carry "<leader>'s group, N of 6". Also this session: confirmed
  the Build .6/.7 flagging works identically for **non-saga containers**
  (live-verified: Harbor Standalones tinted orange with "☠ R5" and the
  Arachnophobia row glowing — saga and non-saga share renderSagaCard/
  renderQuestRow). Verified headless (jsdom, 64 checks, ALL PASS): 0
  runtime errors, self-check unchanged (✓ 62 containers / 517 quests /
  100% detailed), build stamp `07052026.8`; tag shows "Henk · 3/6"
  (leader + 2 members), an anonymous leader shows "Anonymous · 1/6", a
  full group shows "Sunalriz · 6/6", the quest badge carries "R4 · Henk
  3/6", and every Build .1–.7 behavior still passes. Live file:
  `sooks-saga-scroll-07052026-8.html`. Park retention: kept
  `sooks-saga-scroll-07052026-8.html` + `sooks-saga-scroll-07052026-7.html`
  + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-6.html` on 2026-07-05.

  **Build 07052026.7 — reaper-tier signal amplified: pronounced bar tint,
  header R tag, matching quest-row glow (no schema change, still v12).**
  Three changes riding the Build .6 skulls index. (1) The container-bar tint
  is MUCH more pronounced: a strong orange/red color wash INSIDE the bar
  (`background` on `.lfm-mid` / `.lfm-high`, not just border+glow), hotter
  border colors (`#ff8c1a` / `#ff2f1f`), wide halo shadows, and BOTH tiers
  now pulse (`lfmMidPulse` 2.2s / `lfmHighPulse` 1.4s, gated behind
  `prefers-reduced-motion`). (2) A small **header tag** (`.lfm-skull-tag`,
  "☠ R<n>") sits beside the progress bar naming the declared tier — shown
  for ANY declared count: neutral parchment chip for R1–R3 (`.low`), hot
  orange (`.mid`) R4–R7, hot red pulsing (`.high`, `lfmTagPulse`) R8–R10.
  (3) Inside an opened container, the exact quest row with the declared-R
  LFM glows **in the same fashion**: `.quest.lfm-mid-row` /
  `.quest.lfm-high-row` add the color wash plus an inset left edge (inset
  box-shadow, not a real border, so row layout doesn't shift); the red row
  pulses (`lfmRowPulse`). Row classes ride `renderQuestRow`'s existing
  `lfmHit` lookup, so they refresh with the 30s tick. Verified headless
  (jsdom, 60 checks, ALL PASS): 0 runtime errors, self-check unchanged
  (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.7`;
  R4 → orange bars + "☠ R4" mid tag + opening the container glows exactly
  the Saltmarsh quest row orange; R9 → red bars + "☠ R9" high tag + red
  row, orange gone; R2 → no tint/row glow but a neutral "☠ R2" tag; wash +
  hot-border + both-pulse + tag/row CSS assertions hold; live-refresh and
  all Build .1–.6 behaviors intact. Live file:
  `sooks-saga-scroll-07052026-7.html`. Park retention: kept
  `sooks-saga-scroll-07052026-7.html` + `sooks-saga-scroll-07052026-6.html`
  + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-5.html` on 2026-07-05.

  **Build 07052026.6 — reaper-tier LFM highlights + 30-second live refresh
  (no schema change, still v12).** Two changes. (1) Reaper-tier highlights:
  DDO doesn't broadcast skull counts — the LFM `difficulty` field is just
  "Reaper" — but leaders type them into the free-text comment (live examples:
  "lamor r1", "Eveningstar stuff on R1"). `refreshLfmIndex` now parses
  R1–R10 from each comment (`\br\s?(10|[1-9])\b`, case-insensitive, trusted
  regardless of the difficulty flag since live data shows reaper runs posted
  under Normal) and keeps the highest per quest (`skulls` on the index
  entry, folded into the repaint signature). Each container's progress bar
  tints by the highest declared tier among its quests: **R4–R7 = brilliant
  ORANGE** (`.lfm-mid`), **R8–R10 = brilliant RED** (`.lfm-high`, pulsing
  via `lfmHighPulse`, behind `prefers-reduced-motion`); R1–R3 / undeclared =
  no tint. The tint is declared after `has-progress`/`all-done` so the
  hotter glow wins; the per-quest 🔥 badge and tooltips surface the tier
  ("🔥 2 LFM · R8"). Cross-tier name matching (both Saltmarsh containers
  light for one LFM) is by design — same index the badges use. (2) 30-second
  live refresh: `LIVE_REFRESH_MS` interval gated by the Page Visibility API
  — hidden tab = no traffic, `visibilitychange` fires an instant catch-up
  tick. Each tick force-expires the in-memory live caches (`_ddoCache`,
  `_guildCache`, LFM window) and re-renders medallions, guild corner, and
  the LFM index; the signature guard keeps idle ticks from churning the
  DOM. Polling chosen because DDO Audit exposes no push channel (its own
  site polls; API updates ~every 20–30s). Static wiki data and 7-day ref
  caches are untouched. Verified headless (jsdom, 52 checks, ALL PASS):
  0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests /
  100% detailed), build stamp `07052026.6`; "r4" comment parses to
  skulls=4 and tints orange (no red), "R9" flips to red (no orange), "r2"
  clears both, CSS colors + reduced-motion pulse assertions hold, a manual
  `liveRefreshTick()` force-refetches LFMs + character/guild data, and
  every Build .1–.5 behavior still passes. Live file:
  `sooks-saga-scroll-07052026-6.html`. Park retention: kept
  `sooks-saga-scroll-07052026-6.html` + `sooks-saga-scroll-07052026-5.html`
  + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-4.html` on 2026-07-05.

  **Build 07052026.5 — "Anonymous" roster label + brilliant progress bars
  (no schema change, still v12).** Two changes. (1) Anonymous guildmates:
  DDO Audit blanks a member's `name` while their in-game Anonymous flag is
  on (confirmed live against the API — the single-character endpoint even
  returns 403 for anonymous characters; prompted by guildmate "Henk" on
  Shadowdale / Order of the Silver Dragons appearing as "?"). The roster now
  carries an `anon` flag in `_summarizeGuild` (`is_anonymous === true` OR
  blank name) and renders those rows as a faded-italic **"Anonymous"** label
  (`.gm-anon`, with an explanatory tooltip) instead of the old `"?"`
  fallback — level/class stay at full strength since the API doesn't redact
  them. (2) Brilliant progress bars: a saga / non-saga container's header
  bar now lights up when it has ANY progress — `has-progress` class when
  `done > 0` adds a gold border, soft outer glow, a richer green→gold fill,
  and a slow illuminated sheen sweeping the filled portion (animation is
  behind `prefers-reduced-motion: no-preference`); `all-done` when
  `done >= total` glows warmest with a fully gold-tipped fill. Empty bars
  keep the quiet parchment look so in-progress packs pop at a glance.
  Verified headless (jsdom, 44 checks, ALL PASS): 0 runtime errors,
  self-check unchanged (✓ 62 containers / 517 quests / 100% detailed),
  build stamp `07052026.5`; an anonymous fixture member renders "Anonymous"
  with tooltip + intact `Monk · Lvl 7` meta and no `"?"` fallback remains in
  source; fresh state shows zero brilliant bars, marking one quest complete
  lights exactly that saga's bar while others stay quiet, the gold/glow +
  reduced-motion CSS assertions hold, and every Build .1–.4 behavior still
  passes (⚔ quest / 📍 zone / uniform thickness / legibility / fold +
  meta-merge / overflow safety). Live file:
  `sooks-saga-scroll-07052026-5.html`. Park retention: kept
  `sooks-saga-scroll-07052026-5.html` + `sooks-saga-scroll-07052026-4.html`
  + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-3.html` on 2026-07-05.

  **Build 07052026.4 — the guild card folds up (no schema change, still
  v12).** Clicking the guild-name header (or Enter/Space — it's
  `role="button"` with `tabindex="0"`) toggles the roster: folded, the card
  collapses to just the header bar — caret ▸ (open shows ▾), guild name and
  online count still visible, **no `<tbody>` rendered at all**, and the
  quest/area ref resolution is skipped while folded. The fold state persists
  across reloads in the **META localStorage key** (`sooksSagaScrollMeta`,
  outside `state` — no schema bump, Export payload untouched) via new
  merge-style `_readAppMeta()` / `_writeAppMeta()` helpers; the Export
  handler's `lastExportedAt` write was converted from a raw overwrite to
  `_writeAppMeta`, so the fold flag survives an export (previously the meta
  object would have been clobbered). New CSS: `th.gm-fold` (pointer cursor,
  hover brightness), `.gm-caret`. Verified headless (jsdom, 34 checks, ALL
  PASS): 0 runtime errors, self-check unchanged (✓ 62 containers / 517
  quests / 100% detailed), build stamp `07052026.4`; click folds (no tbody,
  ▸ caret, header intact), click reopens (both member cells back), fold +
  unfold each persist to the meta key, meta writes merge (guildFolded
  survives a lastExportedAt write; exactly one raw `setItem` on the meta key
  remains, inside `_writeAppMeta`), and every roster behavior from Builds
  .1–.3 still passes (⚔ quest / 📍 zone / uniform thickness / legibility /
  overflow safety). Live file: `sooks-saga-scroll-07052026-4.html`. Park
  retention: kept `sooks-saga-scroll-07052026-4.html` +
  `sooks-saga-scroll-07052026-3.html` + cross-day anchor
  `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-2.html` on 2026-07-05.

  **Build 07052026.3 — guild-box legibility pass (CSS only, no schema change,
  still v12).** The roster text was EB Garamond at 0.7–0.74rem in faded ink
  with italics — hard to read at the card's size, per user feedback. Body
  text bumped 0.74 → **0.85rem**; the class/level meta and the quest/zone
  line bumped 0.7 → **0.8rem**; faded ink (`--ink-faded`) replaced with
  full-strength `--ink` on both member lines (only the "No guildmates online"
  empty note stays faded); italics dropped from the quest/zone line; the
  Cinzel header bumped 0.7 → 0.76rem; line-heights (1.25/1.3) and row padding
  (3 → 4px) opened up slightly. The `.gm-quest` reserved min-height tracks the
  new size (`calc(0.8rem * 1.3)`), preserving Build .2's uniform row
  thickness. No markup or JS change; all Build .1 overflow-safety rules
  retained (table-layout fixed, no nowrap, overflow-wrap ×3, flex-wrap).
  Verified headless (jsdom, 25 checks, ALL PASS): 0 runtime errors,
  self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build
  stamp `07052026.3`, all prior roster behaviors intact (⚔ quest / 📍 zone /
  uniform thickness / class-first meta), and the five new legibility
  assertions hold (0.85rem body, 0.8rem ×2, faded ink only on gm-empty, no
  italic on the quest line, min-height at the new size). Live file:
  `sooks-saga-scroll-07052026-3.html`. Park retention: kept
  `sooks-saga-scroll-07052026-3.html` + `sooks-saga-scroll-07052026-2.html` +
  cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned
  `sooks-saga-scroll-07052026-1.html` on 2026-07-05.

  **Build 07052026.2 — roster line 2 always renders (uniform row thickness)
  and shows the ZONE when the member isn't in a quest (no schema change,
  still v12).** Two refinements to the Build .1 two-line roster, both user
  requests. (1) The second line now ALWAYS renders and reserves one line of
  height (`min-height: calc(0.7rem * 1.25)` on `.gm-quest`), so every member
  row is the same thickness whether or not their location is known — a member
  with no resolvable location gets a blank-but-reserved line instead of a
  thinner row. (2) When the member is NOT inside a quest instance, line 2 now
  shows their zone — public spaces included — as "📍 <zone>" (extra class
  `.gm-zone`, tooltip "Current zone"), resolved from the areas ref table via
  `getAreasMap()` (same 7-day `sooksSagaScrollRef` cache, already fetched for
  the medallion's live-location feature). Inside a quest it stays "⚔ <quest>"
  (no `.gm-zone`). Both ref tables resolve in a single `Promise.all` with a
  stale-guard after the await. Verified headless (jsdom, 21 checks, ALL
  PASS): 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests
  / 100% detailed), build stamp `07052026.2`, quest member shows ⚔ (not
  `.gm-zone`), public-area member shows "📍 The Marketplace" with `.gm-zone`,
  BOTH cells carry a line-2 div, min-height present, and all Build .1
  overflow-safety assertions still hold. Live file:
  `sooks-saga-scroll-07052026-2.html`. Park retention: kept
  `sooks-saga-scroll-07052026-2.html` + `sooks-saga-scroll-07052026-1.html` +
  cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); nothing
  pruned this park.

  **Build 07052026.1 — guild-roster rows are two stacked lines: name · heroic
  class · level, then the quest that member is running (no schema change,
  still v12).** Each online guildmate in the floating roster now renders as ONE
  full-width cell (`.gm-cell`; the table is single-column now — header `th`
  lost its `colspan`) holding two lines: line 1 (`.gm-line1`, a wrapping flex
  row) = bold name (`.gm-name`) left + `.gm-meta` right, with the meta
  reordered class-first per user request — `Fighter/Monk · Lvl 34` (heroic
  classes only; Epic/Legendary/Destiny pseudo-classes still dropped). Line 2
  (`.gm-quest`, italic, indented, ⚔ prefix) appears ONLY when that member's
  `location_id` resolves to a quest instance — the quest they're running,
  live from DDO Audit. Resolution is a new quest-by-area index: the quests
  reference cache (localStorage `sooksSagaScrollRef`, 7-day TTL, outside
  `state`) now also stores each quest's `area_id` (slim key `area`); caches
  written by older builds lack the key and are treated as stale + refetched
  once (shape guard). New `_questAreaMap` / `getQuestAreaMap()`;
  `_summarizeGuild` keeps each online member's `location_id`. Fit guarantee:
  everything wraps inside the fixed 200px card — `table-layout: fixed`, zero
  `white-space: nowrap` in the guild CSS, `overflow-wrap: anywhere` on name,
  meta, AND quest line, and `.gm-line1` uses `flex-wrap` so the meta drops
  under a long name instead of pushing past the right edge. Verified headless
  (jsdom, 17 checks, ALL PASS): 0 runtime errors, self-check unchanged (✓ 62
  containers / 517 quests / 100% detailed), build stamp `07052026.1`, header
  "⟨The Zhentarim⟩ · 2 online" single-column, self + offline excluded, meta
  `Fighter/Monk · Lvl 34`, quest line renders for a member inside a quest
  area and is absent for a member in a public area, the stale-shape cache
  triggers exactly one refetch and the rewritten cache rows carry `area`, and
  all four overflow-safety CSS assertions hold. Live file:
  `sooks-saga-scroll-07052026-1.html`. Park retention: kept
  `sooks-saga-scroll-07052026-1.html` + `sooks-saga-scroll-06302026-1.html`
  (the latter is also the cross-day rollback anchor, so the retain set is 2
  files); pruned `sooks-saga-scroll-06222026-4.html` on 2026-07-05.

  **Build 06302026.1 — guild-roster table no longer clips names at the right
  edge (CSS only, no schema change, still v12).** The floating online-guildmates
  table (`.guild-table` inside `#guildCorner` / `.guild-panel`) had both
  `.gm-name` and `.gm-meta` set to `white-space: nowrap`. In a fixed 200px card
  whose panel uses `overflow-x: hidden`, a long member name plus a multiclass
  meta string (e.g. `Lvl 34 Fighter/Monk`) overran the box and the right edge
  was clipped. Fix: `.guild-table` now uses `table-layout: fixed` so the two
  columns honor the card width; `.gm-name` is given `width: 52%` and both
  `.gm-name` and `.gm-meta` switched to `white-space: normal` with
  `overflow-wrap: anywhere` + `word-break: break-word`, so long names and
  multiclass metas wrap onto a second line inside the box instead of being cut
  off. No markup or JS change — `renderGuildCorner()` is untouched. Verified
  headless (jsdom): page boots with **0 JS errors**, build stamp reads
  `06302026.1`, `table-layout: fixed` + both cells wrapping confirmed present,
  `renderGuildCorner` still a function. Live file:
  `sooks-saga-scroll-06302026-1.html`. Park retention: kept
  `sooks-saga-scroll-06302026-1.html` + `sooks-saga-scroll-06222026-4.html`
  (the latter is also the cross-day rollback anchor, so the retain set is 2
  files); pruned `sooks-saga-scroll-06222026-3.html` and
  `sooks-saga-scroll-06212026-8.html` on 2026-06-30.

  **Build 06222026.4 — contextual level-relative "Show" reveals both saga AND
  non-saga content (no schema change, still v12).** The level-aware chip used to
  filter to a single tier; now it shows the whole **level band** — the saga tier
  paired with its non-saga sibling: Heroic + Non-Saga Heroic (L1–19), Epic +
  Non-Saga Epic (L20–29), Legendary + Non-Saga Legendary (L30+). Implemented as a
  `band:<tier>` value for `state.tierFilter`: a new `TIER_BANDS` map plus
  `bandTiers()` / `tierFilterActiveSet()` helpers. The saga filter shows every
  member of the band; **both** corresponding buttons in the segmented tier
  control read as active; and `VALID_TIER_FILTERS` now accepts the three band
  values so a persisted contextual show survives a reload (instead of resetting
  to "all"). The chip text reads "suggested: <Tier> + Non-Saga <Tier>" and its
  button "Show <Tier> (saga + non-saga)"; it still folds up all sagas (Build .3)
  for a tidy collapsed overview. Clicking any single-tier button afterward still
  narrows to exactly that one tier. Verified headless (jsdom): `bandTiers("band:Epic")`
  → `["Non-Saga Epic","Epic"]`; a level-25 character's "Show Epic (saga +
  non-saga)" sets `tierFilter="band:Epic"`, the visible cards are exactly Epic +
  Non-Saga Epic, both those buttons are active, all sagas folded (0 open), then a
  single Epic-button click narrows to just Epic; data self-check unchanged (✓ 62
  containers / 517 quests / 100% detailed), 0 runtime errors, build stamp
  `06222026.4`.

  **Build 06222026.3 — corner split into two distinct cards + fold-up on the
  level-relative tier (no schema change, still v12).** Two UX changes.
  (1) The upper-right floating cluster (`.data-corner`) is no longer a single
  box; it's now a transparent wrapper that stacks **two clearly-distinct cards**
  with a gap between them. The **Characters card** (`.corner-panel.char-panel`)
  holds a small "Characters" caption, the character-chip list (`#charCorner`),
  AND the **Export / Import** buttons — so the data actions read as plainly
  belonging to the character roster. The **Guild card**
  (`.corner-panel.guild-panel`, was `#guildCorner`) is a separate box holding the
  live online-guildmates table (its own sticky header is the guild name). Both
  cards share a fixed 200px width so they align; the guild card is hidden when
  empty. (2) The level-aware **"Show <Tier>" button now folds up all sagas** —
  it turns Expand-all off and clears the expand/collapse sets, so showing quests
  for the character's relative level yields a tidy collapsed overview the user
  can expand selectively. New CSS: `.corner-panel`, `.char-panel`, `.corner-cap`,
  `.guild-panel` (replacing `.guild-corner`); the mobile media query moved its
  padding from `.data-corner` onto `.corner-panel`. Verified headless (jsdom):
  the cluster renders exactly two cards in order Characters → Guild, Export +
  Import live inside the Characters card, the guild table is a separate card with
  header "⟨The Zhentarim⟩ · 2 online", and clicking "Show Epic" for a level-25
  character sets the tier filter to Epic with Expand-all off, both expand sets
  empty, and 0 open saga cards; data self-check unchanged (✓ 62 containers / 517
  quests / 100% detailed), 0 runtime errors, build stamp `06222026.3`.

  **Build 06222026.2 — guildmates online moved into a floating roster table
  (no schema change, still v12).** The Build .1 per-medallion "🛡 N guildmates
  online" line is replaced by a proper **floating table** in the upper-right
  cluster (`.data-corner`), sitting directly **under the character-chip list**
  (`#charCorner`) and above the Export/Import row. New markup `#guildCorner` +
  new `renderGuildCorner()` (called from `renderAll`, and from `updateCharStats`
  when the active medallion refreshes). The **table header is the active
  character's guild name** — "⟨<Guild>⟩ · N online" — and each body row lists an
  online guildmate's name (`.gm-name`) and `Lvl L Class` (`.gm-meta`, heroic
  multiclass shown e.g. "Lvl 34 Fighter/Monk"); self and offline members are
  excluded. The header is `position: sticky` so it stays put while the list
  scrolls (`max-height: 40vh`, internal scroll), matching the 180px chip column
  width. Empty/hidden (`:empty`) until the active character has a server set and
  DDO Audit reports a guild. Reuses the Build .1 live-data path
  (`fetchCharacter`→`guild_name`, `fetchGuildmates`→roster; both array and
  object `data` shapes handled), best-effort/async with stale guards, degrades
  silently when unreachable. New CSS: `.guild-corner`, `.guild-table`,
  `.gm-name`, `.gm-meta`, `.gm-empty`. Verified headless (jsdom): the table
  renders under `#charCorner` inside `.data-corner` with header
  "⟨The Zhentarim⟩ · 2 online", rows Kaeithra/Sparkeee (self + offline
  excluded), sticky header, the old medallion `.guildmates` line is gone (0),
  hides when the active char has no server, the data self-check is unchanged
  (✓ 62 containers / 517 quests / 100% detailed), 0 runtime errors, build stamp
  `06222026.2`.

  **Build 06222026.1 — five new DDO Audit live-data features (no schema
  change, still v12).** A research pass against the DDO Audit OpenAPI spec
  (`api.ddoaudit.com/docs/openapi.json`) and live payloads from the new
  servers (Cormyr / Moonsea / Shadowdale / Thrane carry live data; the older
  servers return empty, so the features simply stay inert there) confirmed the
  real shapes, then five features were layered on top of the existing
  single-character live line — all best-effort, offline-first, CORS-safe, and
  keyed off the **active** character's server (`serverOf(state.activeChar)`),
  with **no new persisted state** (schema stays v12; Export/Import round-trips
  unchanged). Reference tables (areas, quests) are cached in a SEPARATE
  localStorage key `sooksSagaScrollRef` (7-day TTL, outside `state` so the
  schema is untouched); live LFM/guild data caches ~60s in memory like
  `_ddoCache`. The five:
  (1) **Live location** — an online character's numeric `location_id` is
  resolved via `GET /v1/areas` and appended to the medallion as "· 📍 <area>";
  if the area name matches a tracked container's name/pack/region (conservative
  substring, no quest-name fuzzing) the card gets a pulsing `saga-now-playing`
  border and a dismissible "📍 <char> is in <area> — part of <saga>" banner
  with a jump-to link.
  (2) **Level-aware tier suggestion** — reuses the persisted v12
  `characterLevels`; a chip by the tier filter reads "⚔ <char> · Lvl N —
  suggested tier: <Tier> (N to <next>)" (1–19 Heroic, 20–29 Epic, 30+
  Legendary) with a "Show <Tier>" button that drives `state.tierFilter`.
  (3) **Last seen** — offline lookups (`source:"database"`, `is_online:false`)
  now read `last_update` (fallback `last_save`) and render "Offline · seen 3
  days ago" via a new `relativeTime()`; no-timestamp offline stays "Offline
  mode".
  (4) **Guildmates online** — `GET /v1/characters/by-server-and-guild-name/
  {server}/{guild}` (handles both array and object `data` shapes) counts online
  members minus self → "🛡 N guildmates online" on the active medallion, tooltip
  lists up to 8.
  (5) **Live LFM badges** — `GET /v1/lfms/{server}` + `GET /v1/quests`
  (quest_id→name) build an index (skipping `quest_id:0`); each quest row whose
  `normalizeQuestName()` matches an open group gets a "🔥 N LFM" badge
  (tooltip: count · level range · difficulties). The async index fetch is
  guarded by a 60s window + change-signature so re-renders can't loop (verified:
  4 renders → 1 fetch). New CSS: `.char-loc`, `.guildmates`, `.quest-lfm`,
  `.tier-suggest`, `.loc-banner`, `.saga-now-playing`. Verified headless
  (jsdom): page boots with the network down with **0 runtime errors**, the data
  self-check is unchanged (**✓ 62 containers / 517 quests / 100% detailed**),
  schema is still v12 and Export→Import round-trips with no new fields,
  location resolves ("Tempest's Spine"), tier mapping + to-next math correct,
  last-seen renders, guildmates counts exclude self, the LFM badge appears only
  on matched quests, and the build stamp reads `06222026.1`.

  **Build 06212026.8 — medallion meta shows heroic classes only (no schema
  change, still v12).** DDO Audit's `classes` array lists "Epic" and "Legendary"
  as pseudo-class entries alongside the real classes (e.g. `Fighter 20 / Epic 10
  / Legendary 2`). `describeCharacter` now drops `Epic` / `Legendary` / `Destiny`
  entries so the class breakdown is just the heroic classes — the aggregate
  **total** level (unchanged) already conveys the epic/legendary span, so the
  rest is inferable. One-line class filter; JS only. Verified headless (jsdom, 7
  checks): Epic/Legendary entries removed, single- and multi-class heroic
  breakdowns preserved, total level untouched. Example line now reads
  `Shadowdale · Lvl 32 · Human Fighter 20 · ⟨Guild⟩`.

  **Build 06212026.7 — medallion polish + offline level persistence (schema
  v11 → v12).** Four changes: (1) the live line now leads with the **aggregate
  (total) character level** — `total_level` if present, else the sum of class
  levels. (2) Every character medallion is now a **fixed size** (width 200px;
  name clamped to one line, the stats line a fixed two-line height with
  ellipsis) so long metadata can't spill and a medallion with no metadata is the
  same size as one with it. (3) A character with no server set now shows
  **"Missing server"** (previously: no line) instead of "Offline mode". (4) New
  additive per-character **`characterLevels`** map (v12) that captures the
  aggregate level from each lookup — online, or last-known from the DB while
  offline — and **persists** it, so future level-specific features keep working
  offline. Additive/backwards-compatible (mirrors `characterServers`), synced on
  add/rename/delete, round-trips via the whole-state Export. JS/CSS only beyond
  the additive field. Verified headless (jsdom, 15 checks): schema v12, aggregate
  level from total or class-sum, level persisted on online and offline lookups,
  "Missing server" rendered, and every medallion carries a stats line for
  uniform sizing.

  **Build 06212026.6 — DDO Audit endpoint corrected (no schema change, still
  v11).** Build .5's self-diagnostic proved `api.ddoaudit.com` is reachable and
  CORS-open (it returned a real HTTP 404, which a CORS block can't do) — the
  path was simply wrong. Confirmed against the backend source
  (`Clemeit/ddo-audit-service`, `sanic/endpoints/characters.py`): the `characters`
  blueprint is `version=1`, so the real base is **`/v1`**, and there's a direct
  single-character endpoint **`GET /v1/characters/<server>/<name>`** → `{ data:
  <character>, source }` (`source:"database"` ⇒ found but offline), with `404`
  for not-found and `403` for anonymous. Rewrote the lookup to call it directly
  (proper- then lower-case server) — no more scanning a whole server list — and
  to render from the real snake_case model (`total_level`, `classes[{name,level}]`,
  `race`, `guild_name`, `server_name`, `is_online`, `is_anonymous`). New distinct
  **"Anonymous"** state for characters hidden in-game; the self-diagnosing
  "Offline mode" tooltip is retained (now also "found but not online" and "not
  found — check spelling"). JS only. Verified headless (jsdom, 12 checks): the
  exact `/v1/characters/{server}/{name}` URL, an online 200 renders the full
  line, an offline `is_online:false` / 404 / 403 / CORS each map to the right
  state, and the medallion wires up end-to-end. **Servers** (13, from the backend
  `SERVER_NAMES`): Argonnessen, Cannith, Ghallanda, Khyber, Orien, Sarlona,
  Thelanis, Wayfinder, Cormyr, Moonsea, Shadowdale, Thrane, Hardcore.

  **Build 06212026.5 — DDO Audit lookup fix + self-diagnosis (no schema
  change, still v11).** A logged-in character was still showing "Offline mode".
  Root cause: the rewritten DDO Audit backend serves live characters at
  `api.ddoaudit.com/characters/{server}` (the public `/who` page), but Build .4
  only queried the old `/players/{server}` path. Fixes: (1) `/characters/{server}`
  is now the primary URL (then `/players`, then legacy `playeraudit.com`);
  (2) the parser also accepts `characters` / `data.characters` response shapes;
  (3) "Offline mode" now **self-diagnoses** — hover it (or open the browser
  console) to see each URL's outcome: `HTTP 404`, `blocked by browser (CORS)`,
  `reached but shape not recognized`, or `reached N online but name not found`.
  That distinguishes the three real causes: wrong endpoint (fixable), a CORS
  block (a `file://` page can't read a third-party API that doesn't allow
  any-origin requests — not fixable in the single file), or a name typo. JS
  only; schema and data unchanged. Verified headless (jsdom, 13 checks): the
  `/characters` shape with classes/guild/location renders the full live line,
  and HTTP-404 / CORS-throw / no-match each produce the correct self-diagnosing
  tooltip.

  **Build 06212026.4 — DDO Audit live character status (schema v10 → v11).**
  Optional, non-blocking enhancement; the tracker stays fully offline-capable.
  New additive per-character `characterServers` map (`{ "<char>": "<server>" }`),
  mirroring `characterThemes`; it round-trips through the existing whole-state
  Export with no extra wiring. Pick a character's DDO server in **Manage** (new
  dropdown) and the medallion gains a live line — `Server · Lvl N · Classes ·
  ⟨Guild⟩ · Location` — pulled from DDO Audit when that character is **online**.
  When the character isn't online, the name doesn't match, or the API can't be
  reached (offline / CORS), the line reads **"Offline mode"**. No server set =
  no live line (medallion unchanged, so existing users see no difference until
  they opt in). This is the app's only network call; the lookup queries a
  server's live player list (`api.ddoaudit.com`, with legacy `playeraudit.com`
  fallback) and matches the name — there is no "find by name" endpoint, per the
  official DDO Audit Discord bot source. The endpoint + response parsing are
  isolated in `DDO_AUDIT.urls` / `parsePlayers` for easy adjustment if the live
  API changes. **Caveat:** whether a browser can reach the API from a `file://`
  page depends on the API's CORS policy, which couldn't be verified here — if it
  blocks cross-origin requests, the line will always show "Offline mode" (a
  safe, non-breaking outcome). Verified headless (jsdom, 26 checks): schema is
  v11, `characterServers` survives ensureShape + Export round-trip, the parser
  handles every known response shape, a stubbed live match renders the metadata
  line, and fetch-reject / no-match both fall back to "Offline mode".

  **Build 06212026.3 — tier-filter button order.** Swapped the Legendary
  and Non-Saga Legendary filter buttons so every saga/non-saga pair now reads
  non-saga-first (Non-Saga Heroic → Heroic, Non-Saga Epic → Epic, Non-Saga
  Legendary → Legendary) — previously the Legendary pair was the odd one out
  (saga-first). Filter-button order only; the section render/group order and
  all quest data are unchanged. The search box is unchanged and intact. No JS
  or schema change (still v10).

  **Build 06212026.2 — tier-filter spill guard (CSS only).** The flat
  tier buttons from .1 sit in a flex row; by default flex items can shrink
  below their content width, and with `white-space: nowrap` (no overflow
  handling) a narrow window would squeeze the boxes and let the labels
  bleed past their borders. Fixed by pinning each button to `flex: 0 0 auto`
  (never shrinks below its label width) plus inline-flex centering, so the
  box always hugs its text; the `.tier-segment.bar` now scrolls horizontally
  (`overflow-x: auto`) if the whole row can't fit, rather than squeezing the
  boxes. No JS, data, or schema change (still v10), Import/Export untouched.

  **Build 06212026.1 — tier filter restyled (CSS/markup only).** The
  page-header tier filter was a 520px two-row grid of wax-tablet buttons
  (`.tier-segment.grid`) that wrapped 4-over-3 and read as ragged beside
  the search box. It's now a tidy single row of modern, flat,
  clearly-boxed buttons (`.tier-segment.bar`): each tier is its own crisp
  1px-bordered parchment box (`--parchment-mid` fill, `--parchment-edge`
  border, 4px radius, 32px tall to line up with the search field); the
  active tier is a solid `--wax-red-dark` fill with parchment text; "All"
  is the same flat box, flattened from the old rounded gold pill and set
  apart with an 8px gap. Markup: container class `grid`→`bar`, buttons
  dropped the unused `grid-btn` class. No JS touched — the tier-filter
  click/active logic keys off `.seg-btn` + `data-tier` + `.active`, all
  unchanged; radio behavior verified. No data or schema change (still
  v10), Import/Export contract untouched. Verified headless (jsdom): 0
  new JS errors, 7 `.seg-btn` boxes, single active tier, clicking a tier
  flips the active box, and the fresh-load empty state is byte-identical
  to Build 06152026.2. Park retention: kept
  `sooks-saga-scroll-06212026-1.html` + cross-day anchor
  `sooks-saga-scroll-06152026-2.html` (also the 2nd-most-recent, so the
  retain set is 2 files); pruned `sooks-saga-scroll-06152026-1.html` and
  `sooks-saga-scroll-06082026-5.html` (removed 2026-06-21).

  **Build 06152026.1 — Cydonie set as the epic Three-Barrel Cove arc
  bestower.** The Epic Pirates of the Thunder Sea container (`e-pirates`)
  modelled its "Pirates of the Thunder Sea (epic) chain — Three-Barrel
  Cove" story arc with `giver: null`. Confirmed via ddowiki (Chrome MCP,
  `ddowiki.com/page/Cydonie`) that **Cydonie** `<Agent of Argonnessen>`
  bestows this chain at **Barrel's Bottom** — her "Bestows Quest(s)" list
  names *Pirates of the Thunder Sea (epic)*, and her Barrel's Bottom
  section states she bestows the chain. Set `giver: "Cydonie"` +
  `giverLocation: "Barrel's Bottom, far south end of the beach, near the
  pier"` on that arc; `renderStoryArc` now renders "— story arc bestowed
  by Cydonie (…)" with an auto-linked wiki page (`renderNpcLink`). The
  **Heroic** Three-Barrel Cove arc (`h-pirates`) is genuine standalone
  quests with per-quest bestowers (Laurent Chastel, Dirty Vingus, …) and
  is **not** on Cydonie's wiki bestows list, so it stays `giver: null`.
  Data-only, schema unchanged (v10), Import/Export contract untouched.
  Verified via jsdom: epic arc giver = "Cydonie" with location, heroic arc
  giver still null, data self-check **✓ 62 containers / 517 quests / 100%
  detailed**, `renderStoryArc` output contains the bestowed-by line + wiki
  link, 0 page errors. Park retention: kept
  `sooks-saga-scroll-06152026-1.html` + `sooks-saga-scroll-06082026-5.html`
  (also the cross-day anchor) + `sooks-saga-scroll-06082026-4.html`; pruned
  `sooks-saga-scroll-06072026-10.html`.

  **Build 06082026.5 — Harbor polish: standalone sort, filter grid,
  conditional Reaper.** Four user-requested changes.
  (1) **Harbor Standalones now sort by heroic level** (ascending, then
  alphabetical within a level): L2 → L3 → L4 → 7 → 10 → 12 → 13 → 14 → 17.
  (2) **Header tier filter restyled as a uniform grid** — every button is
  the same size (equal-width wax tablets in a CSS `auto-fit` grid that
  wraps to a second row), replacing the old flowing ❯❯ chevron strip.
  (3) **Heroic and Non-Saga Heroic flipped** in the filter so Non-Saga
  Heroic comes first (non-saga before saga, matching the Non-Saga Epic /
  Epic pairing). Order is now: Non-Saga Heroic · Heroic · Non-Saga Epic ·
  Epic · Legendary · Non-Saga Legendary · All. (Filter button order only;
  the grouped-list render order is unchanged.)
  (4) **Reaper button removed for solo-only quests** — confirmed against
  the DDO wiki's **Reaper difficulty → "Non-reaper quests"** section (NOT
  inferred). Of the 37 Non-Saga Heroic quests, exactly **5 have no Reaper**
  (all solo-only): **Protect Baudry's Interests**, **Stop Hazadill's
  Shipment**, **Arachnophobia**, **Home Sweet Sewer**, **The Miller's
  Debt**. Added `const NO_REAPER` (a Set of those names); `renderQuestRow`
  omits the ☠ REAPER ☠ button for them, and `sagaStatus` excludes them
  from `ftrRemaining` so a container's Reaper badge can still seal (e.g.
  the Baudry container seals once its one eligible quest, Retrieve the
  Stolen Goods, is Reaper-done). All other 32 Harbor quests keep Reaper.
  UI/data-only, schema unchanged (v10). Verified via jsdom: self-check 0
  errors; filter renders as a grid in the flipped order; standalones
  render in level order; the 5 no-Reaper quests show no Reaper button
  while eligible quests do; Baudry badge seals on the single eligible
  Reaper. Park retention: kept `sooks-saga-scroll-06082026-5.html` +
  same-day previous `sooks-saga-scroll-06082026-4.html` + cross-day anchor
  `sooks-saga-scroll-06072026-10.html`; pruned `sooks-saga-scroll-06082026-3.html`.

  **Build 06082026.4 — Harbor split into per-pack containers + expand/
  collapse fix.** Two changes from user feedback.
  (1) **Per-card collapse now works even when "Expand all" is on.** The
  saga-header click handler used to `return` early whenever
  `state.expandAll` was set, so a card couldn't be tucked away (collapsed)
  while Expand-all was active. Added a `collapsed` Set (mirrors the
  `expanded` Set): when Expand-all is on, `isExpanded = !collapsed.has(id)`
  and a header click toggles `collapsed`; when off, it toggles `expanded`
  as before. The Expand-all button clears both sets on toggle. UI-only,
  no schema change.
  (2) **The single "The Harbor" container is replaced by FIVE per-pack
  Non-Saga Heroic containers**, matching how Non-Saga Legendary / Epic are
  modelled (one card per pack, each independently expandable):
  `ns-h-baudry` (A Man Named Baudry Cartamon, L2, 3q),
  `ns-h-lost-seekers` (The Lost Seekers — Harbor portion, L3–4, 4q),
  `ns-h-harbinger` (Harbinger of Madness, L15, 4q),
  `ns-h-web-of-chaos` (Web of Chaos, L16, 3q), and
  `ns-h-harbor-standalones` (Harbor Standalones, L2–17, 23q, flat list —
  no chain). The four chains keep a single story arc each; the standalones
  container is a flat quest list like `ns-l-free-to-play`. Each chain
  container has a uniform patron (Baudry → The Free Agents, Lost Seekers →
  The Coin Lords, Harbinger → The Free Agents, Web of Chaos → The Silver
  Flame); the standalones container is "Various (free-to-play)" with
  per-quest `QUEST_PATRON_OVERRIDES` (Captive of the Hidden God skipped —
  it appears in another container; the reused Web-of-Chaos/Good
  Intentions/Grim/Sleeping quests resolve detail + patron via their own
  buckets). `QUEST_DETAILS`/`QUEST_GIVERS` re-keyed from the old
  `ns-h-harbor` bucket into the five container buckets; Web of Chaos +
  the three reused standalones resolve via `lookupQuestDetails`
  cross-bucket fallback (heroic level chips confirmed rendering).
  **Note:** container IDs changed, so any progress saved under the old
  `ns-h-harbor` id (builds .2/.3) does not carry into the new containers.
  Schema unchanged (v10). Verified via jsdom: self-check **0 errors / 62
  containers / 517 quests / 517 detailed / 0 missing stories**; all five
  cards render with correct levels, arcs, and per-quest level chips; with
  Expand-all on a header click collapses then re-expands the card. Park
  retention: kept `sooks-saga-scroll-06082026-4.html` + same-day previous
  `sooks-saga-scroll-06082026-3.html` + cross-day anchor
  `sooks-saga-scroll-06072026-10.html`; pruned `sooks-saga-scroll-06082026-2.html`.

  **Build 06082026.3 — fix: Non-Saga Heroic tier rendered nothing.**
  Build .2 added `"Non-Saga Heroic"` to the `TIERS` array and the
  tier-filter `VALID_TIER_FILTERS`/self-check sets, but **missed a second,
  hardcoded tier order inside `renderSagaList()`** (the `groupBy === "tier"`
  branch, `groupOrder = [...]`). The render loop iterates that local
  `groupOrder`, so the Harbor group was built but never drawn — clicking the
  Non-Saga Heroic filter showed "No sagas match the current filter." Added
  `"Non-Saga Heroic"` to that array (slotted after Heroic). Presentation/
  render-only, schema unchanged (v10). Verified via jsdom by dispatching a
  real click on the Non-Saga Heroic button: before = 0 cards + "No sagas
  match"; after = 1 card under a "Non-Saga Heroic" section header with The
  Harbor rendering. **Lesson for future tier additions: there are TWO tier
  orderings to update — the top-level `const TIERS` AND the `groupOrder`
  literal in `renderSagaList()`.** Park retention: kept
  `sooks-saga-scroll-06082026-3.html` + same-day previous
  `sooks-saga-scroll-06082026-2.html` + cross-day anchor
  `sooks-saga-scroll-06072026-10.html`; pruned `sooks-saga-scroll-06082026-1.html`.

  **Build 06082026.2 — new tier: Non-Saga Heroic (The Harbor).** Added a
  sixth tier, **Non-Saga Heroic**, holding one wiki-sourced container,
  **"The Harbor (Stormreach)"** (`ns-h-harbor`, `isNotSaga`), with the
  **37 quests** the DDO Wiki's "The Harbor quests" category lists as
  obtainable in The Harbor (heroic levels **2–17**). Grouped into 5 story
  arcs: the four wiki-recorded chains — **A Man Named Baudry Cartamon**
  (3), **The Lost Seekers** Harbor portion (4), **Harbinger of Madness**
  (4), **Web of Chaos** (3) — plus a **Standalone Harbor quests** bucket
  (23). The Harbor's two saga-tied quests were filed with their sagas
  instead: **Irestone Inlet** added to `h-pirates` (The Pirates of the
  Thunder Sea, Heroic) as a Harbor pre-quest (giver Niles Cage), and
  **Lost at Sea** was already in `h-masterminds`. Every quest carries
  wiki-sourced **story / monsters (with links) / traps / tips / level /
  giver / patron** (Chrome MCP against ddowiki, per quest page; 6 quests
  that already existed in other tiers — Good Intentions, Grim and Barett,
  Servants of the Overlord, The Lords of Dust, The Spinner of Shadows,
  Sleeping with the Fishes — reuse their existing details via
  `lookupQuestDetails` cross-bucket fallback). Patron is a container-level
  field set to "Various (free-to-play)" with per-quest `QUEST_PATRON_OVERRIDES`
  for each Harbor-unique quest (the 6 reused quests are left out of the
  global override to avoid surfacing a patron row in their other
  containers; Captive of the Hidden God likewise skipped — it appears
  elsewhere). **Schema bumped v9 → v10** (additive: one new tier value
  `"Non-Saga Heroic"`; group order is now Heroic → Non-Saga Heroic →
  Non-Saga Epic → Epic → Legendary → Non-Saga Legendary; no stored-state
  field changed, Import/Export contract untouched; older builds reset an
  unrecognized `tierFilter` to "all"). Tier filter gains a Non-Saga Heroic
  button; `tierKey` maps it to `heroic` for the level chip. Verified via
  jsdom: data self-check **0 structural errors**, **58 containers / 517
  quests / 517 detailed / 0 missing stories**, the Harbor card + all 5
  arcs + sample quests render, Irestone resolves under `h-pirates`, no new
  runtime errors. Park retention: kept `sooks-saga-scroll-06082026-2.html`
  + same-day previous `sooks-saga-scroll-06082026-1.html` + cross-day
  anchor `sooks-saga-scroll-06072026-10.html`; nothing pruned.

  **Build 06082026.1 — Druid's Deep arc bestower filled in.** The
  "The Druid's Deep" story arc was modelled with `giver: null` in both
  the **Heroic** and **Epic** Perils of Cormyr sagas, on the prior
  assumption it had no single arc-giver. Confirmed via ddowiki that
  **Anazar Narroun** `<Priest of Amaunatur>` is the arc bestower — his
  NPC page lists "Bestows Quest(s): The Druid's Deep" and is categorized
  under Quest bestowers (the per-quest contacts — Meena Follakin,
  Eolynn Arva, Kirinel Zay, Tarim Inavule — hand out the individual
  quests; Anazar bestows/closes the chain at the northeast edge of
  Eveningstar, just north of the bridge). Set `giver: "Anazar Narroun"`
  + `giverLocation` on both Cormyr "The Druid's Deep" arcs (the Honor of
  the Huntsilvers sagas only carry The Druid's Curse as a standalone
  side quest, so they're untouched). Data-only, schema v9, Import/Export
  contract untouched. Verified via jsdom: both arcs render Anazar, 0
  remaining null Druid's Deep arcs, 0 jsdom errors. Park retention: kept
  `sooks-saga-scroll-06082026-1.html` + cross-day anchor
  `sooks-saga-scroll-06072026-10.html`; pruned
  `sooks-saga-scroll-06072026-9.html` and `sooks-saga-scroll-06032026-2.html`.

  **Build 06072026.10 — shared-quest tag tooltip now works (hover + tap).**
  The `↔ N` shared-quest tag relied on the native `title` attribute, which
  is slow on desktop and invisible on touch (so "nothing happened" on
  hover/click). Replaced it with a **custom CSS tooltip** (dark parchment
  bubble with gold trim) shown on `:hover` and via an `.open` class, and
  wired a **tap/click handler** that toggles the tooltip (closing any
  other) so it works on touch and the click now does something. Render
  uses `data-tip` + `aria-label` (added `role="button"`, `tabindex="0"`)
  instead of `title`. CSS + small JS, no data/schema change (v9). Verified
  via jsdom: 102 tags render with `data-tip`, click toggles `.open`, 0 new
  runtime errors. Park retention: kept `sooks-saga-scroll-06072026-10.html`
  + same-day previous `sooks-saga-scroll-06072026-9.html` + cross-day
  anchor `sooks-saga-scroll-06032026-2.html`; pruned
  `sooks-saga-scroll-06072026-8.html`.

  **Build 06072026.9 — filter tabs restyled (illuminated wax-seal look).**
  The tier filter buttons + the "All" pill were flat parchment rectangles
  and felt plain. Restyled them as embossed, in-theme tablets: parchment
  gradient with a gold (`--gold`) border, a raised top highlight + soft drop
  shadow, uppercase Cinzel lettering with a subtle emboss text-shadow, a
  gold-glow lift on hover, and an **active state that reads like a pressed
  wax seal** — wax-red gradient, gold inner ring, inner shadow, and a soft
  red glow. CSS-only (no markup/JS/data change), schema v9. Verified via
  jsdom: page loads with no new runtime errors. Park retention: kept
  `sooks-saga-scroll-06072026-9.html` + same-day previous
  `sooks-saga-scroll-06072026-8.html` + cross-day anchor
  `sooks-saga-scroll-06032026-2.html`; pruned `sooks-saga-scroll-06072026-7.html`.

  **Build 06072026.8 — removed the "Overall Quest Progress" bar; restored
  per-saga bars.** Corrects build .7, which removed the wrong element. The
  per-saga progress bars on each card are **back**, and instead the
  **"Overall Quest Progress" headline + its big bar** (top of the
  tier-rollup band) is removed. The per-tier breakdown grid is kept. Built
  from .6 to restore the card bars cleanly. Presentation-only, schema v9.
  Verified via jsdom: no "Overall Quest Progress" in render, per-saga bars
  present, 0 self-check errors, no new runtime errors. Park retention: kept
  `sooks-saga-scroll-06072026-8.html` + same-day previous
  `sooks-saga-scroll-06072026-7.html` (the superseded .7) + cross-day
  anchor `sooks-saga-scroll-06032026-2.html`; pruned
  `sooks-saga-scroll-06072026-6.html`.

  **Build 06072026.7 — removed per-saga progress bars (SUPERSEDED by .8 —
  wrong element; reverted).** Each
  saga card's header carried its own thin done/total progress bar, repeated
  across all 57 cards, which made the list look busy. Removed the
  `.saga-progress-bar` block from the card header; the header progress UI
  (top stats band + tier-rollup band) is unchanged, and the reward pill
  still shows each container's Banked/Completed state. Presentation-only,
  no data or schema change (v9); the `.saga-progress-bar*` CSS is now
  unused but left in place (harmless). Verified via jsdom: cards render
  with the bar gone, header bands intact, no new runtime errors. Park
  retention: kept `sooks-saga-scroll-06072026-7.html` + same-day previous
  `sooks-saga-scroll-06072026-6.html` + cross-day anchor
  `sooks-saga-scroll-06032026-2.html`; pruned `sooks-saga-scroll-06072026-5.html`.

  **Build 06072026.6 — filled missing per-quest givers (34 NSL quests).**
  Caught a gap the giver audit missed (it only compared quests that already
  had a giver): **34 Non-Saga Legendary quests had no giver at all (null)**,
  across 9 containers — including the entire **Free-to-Play Standalone
  Legendary** container (10 quests). Set each to its wiki per-quest
  bestower via a `QUEST_GIVERS` patch that creates the container bucket if
  absent: Free-to-Play → Hamish Graymoor, Orben Romblemore, Popkin 'The
  Shiv' Shortshanks, Iris Joll, Frelda Brumlyn, Naleen Lassite, Gavin
  Northus, Tollis Kalmor, Lady Nilith, and **Body and Mind** = Walk-up (no
  NPC, per wiki); plus Dragonblood Prophecy, Hunter and Hunted, Slice of
  Life, Tavern Tales, Devil's Gambit, Soul Splitter, Vale of Twilight, and
  Trials of the Archons. After this, **0 NSL quests are missing a giver**.
  Wiki-sourced via the full 479-quest audit; static data, schema v9.
  Verified via jsdom: 34/34 now populated, 0 self-check errors, no new
  runtime errors. Park retention: kept `sooks-saga-scroll-06072026-6.html`
  + same-day previous `sooks-saga-scroll-06072026-5.html` + cross-day
  anchor `sooks-saga-scroll-06032026-2.html`; pruned
  `sooks-saga-scroll-06072026-4.html`.

  **Build 06072026.5 — per-quest patron exceptions.** Resolved the last
  patron discrepancies from the audit. Patron is a container-level field,
  but a few containers aggregate quests from more than one patron — the
  five **Three-Barrel Cove** quests inside the Pirates/Sentinels container
  and the free-to-play **Lost at Sea** pre-quest inside Masterminds of
  Sharn are wiki **The Free Agents**, not their container's patron. Added a
  small static `QUEST_PATRON_OVERRIDES` map (6 quests) and a Path-block
  **"Patron" row that renders only when a quest's wiki patron differs from
  its container** (so the other 473 quests are visually unchanged); the row
  reads e.g. "The Free Agents (this quest; container patron is House
  Deneith (partial))". Mirrored into the Path edit-template text. Static
  reference data, schema v9. Verified via jsdom: the row appears for all 6
  exceptions and is absent for normal quests; 0 self-check errors, no new
  runtime errors. Park retention: kept `sooks-saga-scroll-06072026-5.html`
  + same-day previous `sooks-saga-scroll-06072026-4.html` + cross-day
  anchor `sooks-saga-scroll-06032026-2.html`; pruned
  `sooks-saga-scroll-06072026-3.html`.

  **Build 06072026.4 — Non-Saga Legendary per-quest givers corrected.**
  The 43 NSL giver differences the audit flagged are now fixed: each NSL
  quest's "given by" is set to its **wiki per-quest bestower** instead of
  the repeated story-arc giver. The **story-arc giver is unchanged** — it
  already lived in each container's `storyArcs[].giver` (Cerra, Darmon
  Kosh, Haden d'Jorasco, Old Tarlow, Una Noldrun, Omryn Cedarbrook,
  Burwick Borley, Tailara Fiercewing, Preceptor Kroth, Asta the
  Deep-Minded), exactly as saga story arcs are modelled — so the chain
  giver and the per-quest bestowers now both display correctly. Applied as
  a post-`QUEST_GIVERS` patch (runs before `buildSharedQuestMap()`) across
  10 containers / 43 quests; wiki-sourced via the full 479-quest audit.
  Examples: Slave Lords quests → Drevok the Gatekeeper; Mines of Tethyamar
  → Hagen Silverthumb / Freygar Plumbline / Eluf the Steadfast / Gunhild
  the Watchful / Inger Ingersdottir. Night Falls on Stormreach (already
  Bril Norgate) untouched. Data-only, schema v9. Verified via jsdom: all
  43 givers match wiki, arc givers intact, 0 self-check errors, no new
  runtime errors. Park retention: kept `sooks-saga-scroll-06072026-4.html`
  + same-day previous `sooks-saga-scroll-06072026-3.html` + cross-day
  anchor `sooks-saga-scroll-06032026-2.html`; pruned
  `sooks-saga-scroll-06072026-2.html`.

  **Build 06072026.3 — patron fix from full wiki audit.** Corrected the
  **Disciples of Rage** container patron from `The Harpers` to **House
  Jorasco** (`ns-l-disciples-of-rage`), matching the wiki and the
  container's own `region` ("Stormreach — House Jorasco") and d'Jorasco
  givers. Found by a full giver/level/patron audit of all 479 quests vs
  ddowiki (288 unique pages): **levels 0 discrepancies; givers all match
  in Heroic/Epic/Non-Saga Epic**; the 43 Non-Saga Legendary giver
  differences are the by-design arc-giver model (each NSL container shows
  the story-arc giver rather than the per-quest bestower) and were left
  as-is per user direction. Remaining patron notes deferred: 5 Deneith-saga
  quests that are wiki "The Free Agents" (carry the "(partial)" aggregation
  marker) and Lost at Sea (wiki "The Free Agents"). Data-only, schema
  unchanged (v9). Verified via jsdom: patron now "House Jorasco", 0
  self-check errors, no new runtime errors. Park retention: kept
  `sooks-saga-scroll-06072026-3.html` + same-day previous
  `sooks-saga-scroll-06072026-2.html` + cross-day anchor
  `sooks-saga-scroll-06032026-2.html`; pruned `sooks-saga-scroll-06072026-1.html`.

  **Build 06072026.2 — Wave 8: doorLocation + NPC location backfill.**
  Data-only, wiki-sourced via Chrome MCP; schema unchanged (still v9,
  Import/Export contract untouched — `doorLocation` and `NPC_LOCATIONS`
  are static reference data, not user state). (1) **doorLocation** for the
  Non-Saga Legendary tier: filled **17 of 86** quests whose ddowiki
  Overview *explicitly* documents the public entrance — the three Slave
  Lords quests (portal in The Gatekeepers' Grove), the four Fall of the
  Night Brigade quests (Wheloon Prison via the Wererats' Entrance), two
  Devil's Gambit, two Lost Gatekeepers (Cerulean Hills), and six Soul
  Splitter (Wharfside Access Lift / Sharn docks, incl. Soul Survivor's
  explicit "Quest Entrance:" note). The remaining **69 stay "Not yet
  sourced"** — their Overviews name only the destination instance, not a
  public door. Per the Wave 4 correction, `doorLocation` is **never**
  inferred from the "Quest acquired in:" / quest-giver area. Applied via a
  post-`QUEST_DETAILS` patch block keyed by `[sagaId, questName]`.
  (2) **NPC_LOCATIONS**: added **10** new wiki-sourced bestower locations
  (Hilda Bitterspine, Wilhelm Finster, Blaze ar'Rhind, Nimbus ar'Yorg,
  Barthold Moncrief, Celia Pendrick, Jer'Can, Zarnoth the Vigilant, Niles
  Cage, Sauriv). **7** NPCs remain unsourced — their wiki pages don't
  exist or carry no Location field (Lord Thekian, Sir Pifflegust,
  Thistledown, Aglaya Karushkin, Davian Martikov, Grilsha Targolova,
  Muriel Vinshaw). (3) **Bug fix:** the Epic Gianthold giver for "Return
  to Madstone Crater" was misspelled `Hworrl ar'Bryssan`; corrected to the
  wiki spelling `Hworrl ar'Brysson`, which now resolves to its existing
  NPC_LOCATIONS entry. Verified via jsdom: 0 self-check errors, 17/86 NSL
  doorLocations present, giver-missing-location count down to 51/445, no
  new runtime errors (the lone navigation-not-implemented notice is the
  pre-existing jsdom limitation). Park retention: kept
  `sooks-saga-scroll-06072026-2.html` + same-day previous
  `sooks-saga-scroll-06072026-1.html` + cross-day anchor
  `sooks-saga-scroll-06032026-2.html`; nothing newly pruned.

  **Build 06072026.1 — backup nudge + schema self-check.** Two small,
  schema-safe additions (still v9, Import/Export contract untouched).
  (1) **"Last exported" footer nudge** — the Export handler now records a
  timestamp in a SEPARATE localStorage meta key (`sooksSagaScrollMeta`,
  outside `state`, so no schema bump and no change to the export payload).
  A new `#lastExport` footer line + `renderLastExport()` shows "Last
  exported N days ago" (a wax-red "No backup yet" warning when none exists,
  and a warning again after 14 days), since all progress lives only in
  localStorage. (2) **Schema-version self-check** — `renderSelfCheck()`'s
  detail report now prints the live `SCHEMA_VERSION` + `STORAGE_KEY` with a
  reminder to confirm the README's "Current shape" heading matches, so the
  v4-vs-v9 doc drift just fixed can't silently recur. Housekeeping this
  build: corrected the README schema block from a stale v4 snapshot to the
  real v9 shape, and retired the separate `CONTEXT.md` (README is now the
  single source of truth). Verified via jsdom: no new runtime errors (the
  lone navigation-not-implemented notice is a pre-existing jsdom
  limitation, present in the prior build too); footer nudge + schema line
  render correctly. Park retention: kept
  `sooks-saga-scroll-06072026-1.html` + cross-day anchor
  `sooks-saga-scroll-06032026-2.html`; pruned
  `sooks-saga-scroll-06032026-1.html` and
  `sooks-saga-scroll-06022026-13.html`.

  **Build 06032026.2 — progress rollups.** Added a per-tier + global
  completion rollup under the existing three-stat progress band. A new
  `#tierRollup` band renders (1) an **Overall Quest Progress** bar with %
  plus `done / total quests` and `complete / total containers`, and (2) a
  responsive grid covering **all five tiers** — each showing
  `containers complete / total`, a thin per-tier fill bar, and a
  `quests done / total · %` subline (turns green when every container in
  the tier is complete). New compute fn `tierRollup(char)` derives
  everything from existing state (`sagaStatus()` per container) on each
  render — **nothing persisted, schema unchanged (v9)**, Import/Export
  contract untouched. `renderProgress()` now also guards the fresh-install
  empty state: with no active character both bands clear and the rollup
  re-hides via the `.is-empty` class. CSS is accent-color-only (the N/M
  counts and % carry meaning, matching the project's text-load-bearing
  pattern); mobile breakpoint tightens the grid to 130px columns.
  Verified via jsdom: rollup renders 5 tiers, math checks (1/57
  containers + 12/479 quests = 3% on a seeded character), container sum
  equals SAGAS.length (57), empty-state guard re-hides, 0 JS errors. Park
  retention: kept `sooks-saga-scroll-06032026-2.html` + same-day previous
  `sooks-saga-scroll-06032026-1.html` + cross-day anchor
  `sooks-saga-scroll-06022026-13.html`; pruned
  `sooks-saga-scroll-06022026-12.html`.

  **Build 06032026.1 — Epic saga reorder.** Swapped the display order of
  two adjacent containers in the **Epic** tier so **The End of Eberron**
  (`e-eberron`) now renders **before Menace of the Underdark**
  (`e-menace`). Implemented by transposing the two objects in the `SAGAS`
  array — no field added, renamed, or removed; both containers keep all
  their data (quests, storyArcs, givers) intact. Epic tier order is now:
  Pirates of the Thunder Sea → **The End of Eberron** → **Menace of the
  Underdark** → Planeswalker's Path → Perils of Cormyr → Honor of the
  Huntsilvers → In the Wastes of Gianthold. Data-only, **schema
  unchanged (v9)** — Import/Export contract untouched. Verified via Node:
  `SAGAS` parses with no JS errors, 57 containers intact, Eberron index <
  Menace index. Park retention: kept `sooks-saga-scroll-06032026-1.html`
  + cross-day anchor `sooks-saga-scroll-06022026-13.html` + third slot
  `sooks-saga-scroll-06022026-12.html`; pruned the older anchor
  `sooks-saga-scroll-06012026-1.html`.

  **Build 06022026.3 (Wave 7) — four more Non-Saga Epic packs.** Added,
  ML-ordered immediately after Keep on the Borderlands (Non-Saga Epic is
  now **7 containers / 30 quests**):

    - **The Red Fens** (ML 21) — 4-quest chain (The Claw of Vulkoor,
      Fathom the Depths, The Last Stand → **Into the Deep** finale, which
      can't be red-doored). Givers in the Raveneye Refuge Camp.
    - **Vault of Night / VoN** (ML 21–22) — VoN 1–5 (Haywire Foundry, The
      Prisoner, The Jungle of Khyber, Tharashk Arena, The Vault of Night).
      Arc giver **Marek Malcanus**. **Excluded:** the **Plane of Night**
      raid (VoN 6) and the heroic-only connector **Gateway to Khyber**
      (no Epic version).
    - **Zawabi's Refuge / Demon Sands** (ML 21–22) — the Epic quests: An
      Offering of Blood, Chains of Flame, The Chamber of Raiyum (flagging,
      any order) → **Against the Demon Queen** pre-raid quest. **Excluded:**
      the **Zawabi's Revenge** raid and the heroic-only lower quests.
    - **White Plume Mountain & Other Tales** (ML 21–25) — the 4 Epic
      side-tales (Kind of a Big Deal, Another Man's Treasure, Memory
      Lapse, The Price of Freedom). Standalone (no chain); patron The
      Twelve. The namesake **White Plume Mountain** quest is Legendary 32
      and remains under the Non-Saga Legendary tier.

  Each quest carries wiki-sourced story / monsters / traps / survival
  (Chrome MCP, per quest page). Data-only, schema unchanged (v9).

  **Build 06022026.4** — shortened that container's display name to just
  **White Plume Mountain** (the `pack` field stays "White Plume Mountain
  and Other Tales"). Display-only.

  **Build 06022026.5** — reordered the Non-Saga Epic section so **The Red
  Fens** (ML 21) sits ABOVE **The Keep on the Borderlands** (ML 21–22).
  New section order: Phiarlan Carnival, Time Flies, The Red Fens, Keep on
  the Borderlands, Vault of Night, Zawabi's Refuge, White Plume Mountain.
  Also re-verified **Vault of Night** quest order (VoN 1–5: Haywire
  Foundry, The Prisoner, The Jungle of Khyber, Tharashk Arena, The Vault
  of Night) — already correct. Order-only, schema v9.

  **Build 06022026.6** — the header **Sagas Complete** and **Rewards
  Banked** counters now count **true sagas only**. Non-Saga Epic and
  Non-Saga Legendary containers (`isNotSaga`) are excluded from both
  totals (the Heroic / Epic / Legendary breakdown already ignored them).
  `charSummary()` returns early on `isNotSaga` before tallying complete /
  banked. Count-only, schema v9.

  **Build 06022026.7 — two usability features.**

    - **Quest/saga search** — a Search box in the controls row filters
      visible cards by saga / pack / region / patron / tier, quest name,
      quest-giver, story-arc name, and each quest's wiki story + monster
      names + survival notes. Space-separated tokens are AND-matched; a
      live match count and a ✕ clear button are shown. In-memory only
      (not persisted, cleared on reload); the haystack is cached per saga.
    - **Data integrity self-check** — runs once on load over the static
      wiki data and shows a footer badge: **✓ Data OK · N containers · M
      quests · X% detailed**, or **⚠ N data issues** (click to expand the
      list; details also `console.warn`-ed). Errors are genuine structural
      problems (duplicate ids; orphan `QUEST_DETAILS` / `QUEST_GIVERS` /
      `SAGA_STORIES` buckets; story-arc quests missing from a container;
      unknown tier). "Not yet sourced" coverage gaps and unsourced giver
      locations are reported as **info**, never errors. Current data: 0
      structural errors, 479/479 quests detailed.

  Both are UI-only, no schema change (still v9).

  **Build 06022026.8 — search polish + behavior.** The search field is now
  a single `.search-field` box (left magnifier glyph, inline ✕ clear, focus
  ring) sharing the `.ctl-input` baseline so it lines up flush with the
  Group-by select and the tier / Expand buttons; the live match-count is
  positioned absolutely beneath the field so it no longer lifts the input
  above the button row. A search now **overrides the Tier filter and spans
  all tiers** — the tier segment dims while a search is active and the count
  reads "N matches · all tiers". UI-only, schema v9.

  **Build 06022026.9 — ranked results (best match first).** While a search
  is active, tier/region/pack grouping is replaced by a single flat
  **"Search results — best match first"** list, sorted by a relevance score
  (`searchScore`): hit quality is exact > prefix/word-boundary > substring,
  weighted by field — name > quest name > giver > story-arc > pack >
  region/patron > tier > detail text — scoring both the whole phrase and each
  token. So "red fens" lands The Red Fens on top, "total chaos" surfaces Keep
  on the Borderlands, "velah" finds Vault of Night. The search index was
  refactored to a per-saga `searchFields` cache. UI-only, schema v9.

  **Build 06022026.10 — tighter narrowing.** Match membership now requires
  every token to **co-occur within a single unit** — a saga / pack / region /
  patron / tier value, a quest name, a giver, or one quest's detail
  (name + story + notes + monsters) — instead of being scattered anywhere in
  the record. So adding words to a query removes items that no longer truly
  match ("vault" → 7 hits; "vault of night" → Vault of Night on top with the
  loose hits gone; "keep borderlands" → just Keep on the Borderlands).
  Single-token searches are unchanged. UI-only, schema v9.

  **Build 06022026.11–.12 — phrase match.** The query is treated as ONE
  contiguous string (word order honored), matched case-insensitively as a
  substring of a unit — so distinctive multi-word names match precisely and
  partials still narrow as you type.

  **Build 06022026.13 — names only.** Match membership is now restricted to
  **saga name, quest name, and NPC (quest-giver / story-arc giver) name**.
  Pack / region / patron / tier and quest detail text (monsters / story /
  notes) are no longer searched, so only containers whose name, quest, or
  NPC contains the typed phrase appear ("velah"/"graz" → no results now;
  "norworth koln" → Keep on the Borderlands; "red fens" → The Red Fens).
  Case-insensitive, whole-phrase. UI-only, schema v9.

  **Build 06022026.2 — collapse-on-complete.** When a quest is marked
  complete and tucked into its saga card's **✓ Completed** rollup drawer
  (any tier), its expanded detail card is now auto-collapsed so the
  drawer stays compact. Implemented in `setSagaDoneFor()` — when it sets
  `sagaDone[q] = true` it also clears `state.questExpanded[sagaId][q]`
  (covers the left checkbox, the Mark-COMPLETE button, and Reaper/Skip
  side-effects, cascading through same-tier siblings). The user can still
  manually re-expand a completed quest inside the drawer. UI/UX only — no
  schema change (still v9).

  **Build 06022026.1 (Wave 6) — added The Keep on the Borderlands** to
  the **Non-Saga Epic** tier (now 3 containers / 13 quests). Epic ML
  21–22, so it sorts to the bottom of the section. 8-quest story arc
  bestowed by **Norworth Koln** `<Castellan>`, ordered: Watch Your Step →
  The Bugbear Bandits → Violent Delights → Treasure Hunt → Obstructing
  the Orcs → Caged Beast → The Hobgoblin Horde → **Total Chaos (finale)**.
  Each quest carries wiki-sourced story / monsters / traps / puzzles /
  survival (Chrome MCP against each quest's ddowiki page). **Crest
  highlight:** the three **Charms of Evil Chaos** — Kobold (*Watch Your
  Step*), Gnoll (*Violent Delights*), Orc (*Obstructing the Orcs*) —
  unlock the **bonus chest + 25% XP** optional in Total Chaos; flagged
  with ★ in each quest's notes and the synopsis. Data-only, schema
  unchanged (v9).
- Filename pattern is `sooks-saga-scroll-<MMDDYYYY>-<N>.html` —
  **kebab-case throughout, no underscores.** See **⚠ Park convention**
  below for retention rules.

## ⚠ Park convention — READ BEFORE EDITING ANYTHING

This project's park rules **override the garage default**. Read this
section before touching any file in this folder.

- **COPY-FIRST, NEVER EDIT IN PLACE.** When the current build is
  `sooks-saga-scroll-<MMDDYYYY>-<N>.html`, the very first step of any
  new build is:

  ```
  cp sooks-saga-scroll-<MMDDYYYY>-<N>.html  sooks-saga-scroll-<MMDDYYYY>-<N+1>.html
  ```

  Then make **all** edits in the new file only. The old build is a
  rollback artifact — overwriting it destroys our ability to roll back.
  If you edit the current build's file directly and then rename it,
  the historical `-N` content is gone. (This rule exists because I
  violated it on Build 05262026.3 and lost the `.1` build.)

- **Filename:** **kebab-case throughout, no underscores anywhere.**
  Pattern: `sooks-saga-scroll-<MMDDYYYY>-<N>.html`
  (e.g. `sooks-saga-scroll-05282026-3.html`). The hyphen between the
  date and the iteration is intentional — this overrides the
  garage-park-retention skill's `_N` example. Both the slug and the
  build-number separator are hyphens here.

- **Retention — 3 builds with cross-day anchor.** Always keep exactly
  3 builds in the directory, selected this way:
    1. The 2 most recent builds overall.
    2. The most recent build from a previous day (if one exists).
       This is the cross-day rollback point — a session that produces
       many same-day builds shouldn't be able to erase yesterday's
       last build in three quick parks.
  If no previous-day build exists yet (project is brand new, or every
  build so far is from today), just keep the 3 most recent. When the
  current set has any file outside the retain set, delete it
  (`mcp__cowork__allow_cowork_file_delete` first).

- **`README.md` and `CONTEXT.md` stay unversioned** — they always
  reflect the latest build.

- localStorage data carries forward across builds — opening a newer
  build picks up wherever the previous build left off. Schema
  migrations are append-only (see contract below).

## Key concepts
- **Reaper / Skip persist when un-completing a quest (Build 06012026.1+).**
  Un-checking the master **Quest Completed** mark — including moving a
  quest back out of the **✓ Completed** rollup into the active list — is
  now a **non-destructive toggle**: it clears only `sagaDone[q]`. The
  First-Time **Reaper** seal (`quests[q] === "reaper"`) and the **Skip**
  mark (`skipDone[q]`) are left intact. They are per-life
  achievements/resources and persist until **↻ True Reincarnation** (which
  zeroes the active character's `quests`) or **⚠ Master Reset**. This fixes
  a bug where un-completing a Reaper quest wiped its Reaper state.
  Two coupled changes made it durable:
    1. `toggleSagaDoneFor()` no longer calls `applyQuestState(qName, false)`
       on the Reaper/Skip path — it only flips `sagaDone`. The old "whole
       row reset" confirm dialog is gone; the explicit reset still lives on
       the **re-click a sealed Reaper button** path (which keeps its
       confirmation).
    2. The v7 `sagaDone` seed and v8 `skipDone` seed in `ensureShape()` are
       now **version-gated** (`fromVersion < 7` / `< 8`). They previously
       re-ran on every load and re-derived `sagaDone[q]=true` from any
       truthy `quests[q]`, which re-stuck the green seal onto an
       un-completed Reaper quest on reload. `migrate()` records the original
       on-disk version in a transient `__fromVersion` (Import falls back to
       the incoming `schemaVersion`); `ensureShape()` deletes it before
       return so it never persists or exports. **No schema bump** — still
       v9, Import/Export contract untouched, and genuine pre-v7/pre-v8 data
       (and old exports loaded via Import) still migrates correctly.
- **Completed-quest rollup (Build 05312026.1+).** Inside each saga card,
  quests carrying the master **Quest Completed** mark (`sagaDone`) are
  pulled out of the main quest list and tucked into a collapsible
  **"✓ Completed (N)"** drawer at the bottom of that saga's quest list —
  cutting on-screen clutter so the active quests stay front-and-center.
  Works for both flat sagas and story-arc sagas: `renderQuestListBody()`
  splits active vs. completed, `renderStoryArc()` now filters out
  completed quests (an arc whose quests are all done renders nothing),
  and `renderCompletedRollup()` builds the drawer by reusing
  `renderQuestRow()` so each tucked quest keeps every control (un-complete,
  Reaper, Skip, detail expand, wiki links). When every quest in a saga is
  done, a short "all tucked into the rollup below" note shows in place of
  the empty list. The drawer is **collapsed by default on every load** —
  open/closed state lives in an in-memory `rollupOpen` Set (mirroring the
  saga-card `expanded` pattern), deliberately **not** a persisted state
  field, so the list always opens tidy and **no schema bump was needed**
  (still v9; Import/Export contract untouched). The header click toggle
  flips the Set + `.open` class directly (no re-render). Only what counts
  as "complete" for the drawer is the Quest Completed master mark —
  Reaper/Skip alone do not tuck a quest away. CSS: `.completed-rollup`,
  `.completed-rollup-header`, `.completed-rollup-body`, `.all-complete-note`.
- **Three tiers, 29 sagas, 353 quest slots.** Heroic (13 sagas), Epic (7),
  Legendary (9). The list mirrors the user's source spreadsheet, with the
  e-pirates saga rounded out to its full 10 quests in `v.05232026.11`
  (the original sheet had only the 5 Three-Barrel Cove quests; the 5
  Sentinels of Stormreach quests — Black Loch, Storm the Beaches, Bargain
  of Blood, The Tide Turns, Spies in the House — were added per the user's
  correction).
- **Characters are mutable.** Default seed is `Xouk` / `Xiin` / `Xokotli`
  but the "✎ Manage" panel lets the user rename, add, or delete characters.
  At least one character is always required (deleting the last is refused).
- **Storage v2.** Shape:
  `{schemaVersion: 2, activeChar, characters, progress, groupBy, ftrOnly,
  expandAll, questExpanded}`. `loadState()` migrates the legacy v1 shape
  (top-level character keys) into the v2 `progress` object on first load;
  the legacy `sooksSagaScroll.v1` localStorage key is read once as fallback.
  Never change the stored shape without adding a matching migration step.
- **☠ Reaper ☠ badges (Build 05242026.5+).** Each unchecked quest wears
  a "☠ REAPER ☠" wax-seal badge — skulls flank the word on both sides and
  render larger than the text (was a single leading skull through Build
  05242026.4; was "FTR" through `v.05232026.7`). A saga's saga-level badge
  clears to "SEALED" only when every quest in it is checked.
- **End Reward toggle is two-state ON/OFF (Build 05242026.5+).** Per-saga
  reward is now `incomplete` (default, off) or `banked` (on) — the legacy
  `not_earned` / `banked` / `redeemed` three-state picker collapsed into a
  single toggle button. When the user checks off the last unchecked quest
  in a saga, the change handler auto-flips reward from `incomplete` to
  `banked` (one-shot — the user can then click the toggle to set it back
  to `incomplete` and it stays off; subsequent quest re-checks won't
  override the user's explicit OFF choice). Migration in `ensureShape()`
  maps `not_earned -> incomplete` and `redeemed -> banked` on first load.
- **Quest completion is 3-state (Build 05242026.6+).** Each quest stores
  one of `false` (not done), `"normal"` (done via the standard checkbox),
  or `"reaper"` (done on first-time Reaper via the clickable ☠ badge).
  Both the checkbox and the badge mark the quest as "done" for saga-progress
  counting — they just record HOW it was done. Click rules:
    • Checkbox: false ↔ "normal" (clicking while reaper-done clears entirely).
    • Reaper badge: false/"normal" → "reaper"; "reaper" → "normal" (downgrade —
      use the checkbox to clear entirely).
  The Reaper badge has three visual variants: solid red wax (challenge available),
  outlined red (you can still claim FTR on a normal-done quest), and gold "SEALED"
  (FTR earned). The checkbox itself gets a red+gold skull-stamp treatment when
  the row is reaper-done. The saga-level `ftrRemaining` count now means
  "quests still not done on Reaper" — so the saga badge stays `☠ REAPER ☠`
  until every quest has been clicked through the Reaper button, only then
  clearing to `SEALED`. Migration converts legacy boolean `true` -> `"normal"`.
- **Skull wrap fix (Build 05242026.6+).** `.ftr-badge` is now `display:
  inline-flex` with `white-space: nowrap` so the flanking skulls never wrap
  under the word "REAPER". Skulls render at 1.25× the text font-size.
- **Completion-controls cluster (Build 05242026.7+).** The standard
  checkbox and the Reaper button now sit inside a single
  `.completion-controls` wrapper at the start of each quest row, framed by
  a dashed parchment border so they read as a paired wax-stamp control
  group. The checkbox was reshaped into a round green-wax stamp (parchment
  outline when empty, emerald fill with a checkmark when checked) sized to
  match the Reaper button's height (22px). When the row is reaper-done,
  the checkbox itself swaps to a red-and-gold skull stamp so the whole
  cluster reads as a single sealed unit.
- **Isle of Dread chain bestowers surfaced (Build 05242026.8+).** The
  earlier audit flagged Curses of the Kopru and Isle of Adventure as
  having no chain giver. That was half-true — each quest in the chain has
  its own bestower, but the wiki documents that they cluster at named NPC
  *camps* in Village of Tanaroa (Curses of the Kopru camp; Hunters' Guild
  camp). `storyArcs[i].giverList: [...]` plus `giverLocation` now carries
  the camp + bestowers. `renderStoryArc()` gained a fourth display case:
  when `giver` is null but `giverLocation` + `giverList` are set, it
  renders "— bestowers gather at &lt;camp&gt;: NPC1, NPC2, NPC3, NPC4"
  with each NPC name linked to its wiki page. Pattern can be replicated
  for any other arc that follows the same "camp of NPCs" model.
- **Story arc bestowers, properly distinguished (Build 05242026.10+).**
  The canonical DDO pattern is 3 tiers of NPCs: **saga giver** (e.g.
  Mattias Deckland for the Heroic Saltmarsh Saga) > **story arc giver**
  (e.g. Eda Owlans for the "Sinister Secret of Saltmarsh (story arc)") >
  **per-quest givers** (Krag, Anders Solmor, Eliander Fireborn,
  Gellan Primewater, Sauriv). Earlier builds were conflating per-quest
  givers with story-arc givers and missing the chain-bestower layer
  entirely. Confirmed arc-givers added this build:
    - Sinister Secret of Saltmarsh (main arc) → **Eda Owlans** (Snapping Line Inn, center)
    - Lord Arden's Slumber → **Gale Estival** (Wynwood Hall, SE)
    - Hyrsam and the Needle → **Hawthorn** (Wynwood Hall, SW upper terrace)
    - The Chill of Ravenloft Part One → **Mayor Vogelhertz** (Ludendorf City Hall)
    - The Chill of Ravenloft Part Two → **Ilona Rabus** (Ludendorf City Hall)
    - Curses of the Kopru → **Mira of the Hawk** (Curses of the Kopru camp, Tanaroa)
    - Isle of Adventure → **Oksi of Tanaroa** (Hunters' Guild camp, Tanaroa)

  The temporary `giverList` field (introduced in 05242026.8 to enumerate
  per-quest NPCs at the camp) was removed — that data was the wrong
  abstraction; the proper bestower fits the existing `giver` field and
  the per-quest NPCs already live in `QUEST_GIVERS`. `renderStoryArc()`
  collapses back to its original 3-case form (named arc giver / side
  quests / no chain). Arcs genuinely without an arc-giver per wiki:
  the **Heroic** Three-Barrel Cove standalones, Wastes of Gianthold,
  Return to Gianthold — these stay `giver: null`. (Druid's Deep left this
  list on Build 06082026.1 — ddowiki confirmed **Anazar Narroun** as its
  arc bestower; both Cormyr "The Druid's Deep" arcs now carry him. The
  **Epic** Three-Barrel Cove chain left it on Build 06152026.1 — ddowiki
  confirmed **Cydonie** `<Agent of Argonnessen>` bestows the epic Pirates
  of the Thunder Sea chain at Barrel's Bottom; the `e-pirates`
  Three-Barrel Cove arc now carries her. The Heroic Three-Barrel Cove
  standalones remain giver-null — they are per-quest bestowed, not a
  Cydonie chain.)
- **Saga `Wiki` row removed (Build 05242026.10+).** Redundant — the
  saga.wiki link added nothing the per-quest, per-NPC, per-arc links
  didn't already provide. `saga.wiki` stays in the data model.
- **Blank-by-default character list (Build 05242026.10+).** New installs
  start with no characters; the header shows a `No characters yet — click
  + Add Character to begin.` empty-state hint and the saga list shows a
  themed prompt. `DEFAULT_CHARACTERS` is now `[]`; `LEGACY_FALLBACK_CHARACTERS`
  preserves the old Xouk/Xiin/Xokotli trio strictly as a v1-import fallback
  so existing users don't lose data. `defaultState.activeChar` and
  `ensureShape` tolerate `null` activeChar. `getSagaState(null, …)` returns
  an empty in-memory sagaState so render and summary code stays branchless.
- **+ Add Character button (Build 05242026.10+).** New emerald-wax button
  in the medallion row, sits flush left of `✎ Manage`. Click opens
  manage mode and focuses the new-character input. Manage button stays
  as the rename/delete affordance.
- **Medallion theme tag removed, Export/Import anchored (Build 05242026.24+).**
  Dropped the `.char-theme-tag` label that was rendering under the
  active medallion ("Rogue" / "Scholar" / etc) — theme is a binding,
  not a visible label per user request. Theme is still in the medallion
  hover title for reference. The unused `.char-theme-tag` CSS rule is
  gone too.

  Fixed: the upper-right Export/Import buttons were being scrolled off
  when the chip list grew (the whole `.data-corner` had `overflow-y:
  auto`, so once chips filled the available height the buttons fell
  below the visible fold). Moved the `overflow-y: auto` from the
  cluster to `.char-corner` alone. Now the chip stack scrolls
  internally if the roster is long, but the `.data-actions` row
  (Export + Import) stays anchored at the bottom of the cluster with
  `flex-shrink: 0` — always visible regardless of how many characters
  are in the list.
- **Color-blind friendly char chips (Build 05242026.23+).** Inactive
  chips switched from a 70%-opacity saturated wax fill to a fully
  transparent background with dark `var(--ink-primary)` text on the
  parchment cluster — high contrast, no reliance on hue. Only the
  **active** chip uses the themed wax color, so the user doesn't need
  to distinguish between saturated reds/greens/blues to find the
  selected character. A small 8px theme-color dot in front of each name
  acts as a low-key visual accent but isn't load-bearing — the chip is
  fully readable from text alone. Hover lifts inactive chips with a
  subtle dark wash + box-shadow instead of a fill swap.
- **Char chips show full names, stacked above Export/Import (Build 05242026.22+).**
  `.data-corner` switched to `flex-direction: column`. The chip row
  re-flowed into a vertical stack of full-name pill chips at fixed
  width (180px desktop, 130px mobile) — each chip is the same padded
  size regardless of name length, so a list of "Bob" / "Yndrofian12345"
  / "Astra" all align. Names that exceed the pill width truncate with
  `text-overflow: ellipsis`; the full name is in the hover title.
  Export/Import buttons moved into a `.data-actions` flex row at the
  bottom so they split the cluster width evenly. Cluster grows
  vertically as more characters are added; capped at `100vh - 24px`
  with internal scrolling for very long rosters.
- **Floating character switcher in top-right (Build 05242026.21+).**
  The viewport-fixed `.data-corner` cluster now leads with a row of
  `.char-chip` round buttons (30×30, character initials in Cinzel bold),
  one per character, separated from Export/Import by a dashed parchment
  rule. Click a chip to switch the active character without scrolling
  back to the header — calls the same code path as the big medallions
  (sets `state.activeChar`, runs `applyTheme()`, re-renders). Each chip
  carries its character's bound theme color via a
  `[data-chip-theme="..."]` attribute selector (Scholar = red wax,
  Divine = sky-blue, Necrotic = bilegreen, Knight = crimson, etc.) so
  users recognize characters by color even when names are abbreviated.
  Active chip is fully opaque + elevated, inactive chips are 70%
  opacity. Empty character list hides the chip row via `:empty`.
- **Theme depth — ornaments, flavor copy, active-theme caption (Build 05242026.20+).**
  Each theme now carries its own header glyphs and subtitle flavor so
  the page reads as that archetype's compendium when the character is
  active. `THEMES` registry gained `ornament` and `subtitle` fields per
  entry; `applyTheme()` swaps `#ornament` and `#subtitle` textContent on
  every theme change:

  | Theme | Ornament | Subtitle line |
  |---|---|---|
  | Scholar | ❦ ✦ ❦ | A wayfarer's compendium |
  | Soldier | ⚔ ★ ⚔ | A campaign ledger |
  | Ranger | ❀ ⚝ ❀ | A trail-bound compendium |
  | Barbarian | ⚒ ☠ ⚒ | A bloodied tally |
  | Rogue | ❖ ✦ ❖ | A pilfered record |
  | Divine | ☉ ✠ ☉ | An anointed reckoning |
  | Arcane | ☽ ✦ ☽ | An arcanist's grimoire |
  | Necrotic | ☠ ✠ ☠ | A reaper's index |
  | Knight | ⚜ ⚔ ⚜ | A herald's roster |

  The **active character medallion** now shows the bound theme label as
  a small Cinzel uppercase caption (`.char-theme-tag`, gold-light on the
  themed wax background), so the user can see at a glance which theme
  the page is currently showing. Non-active medallions stay clean.

  The active medallion's wax background switched from hardcoded
  `#c9342a` to `var(--wax-red-light)`, so it now picks up each theme's
  primary "wax" color — sky-blue for Divine, bilegreen for Necrotic,
  silver for Rogue, etc. The whole medallion reads as part of the theme.
- **Themes actually visible now (Build 05242026.19+).** The previous
  build wired the theme machinery correctly but the dominant visible
  surfaces (`.scroll` parchment background, `.saga` card overlay,
  `.quest-details` expanded card, `.char-manage-panel`, `.saga-header:hover`)
  were all using hardcoded warm-cream rgba values instead of CSS
  variables — so switching themes only changed the body background
  *behind* the scroll, which is largely hidden. Fixed:
    - `.scroll` now uses `var(--parchment-base)` / `mid` / `shadow` in
      its base gradient instead of the literal `#f1e4be` / `#e3d1a4` /
      `#d8c594` trio. Texture overlays (stains, age spots) and the inset
      box-shadow shifted to neutral `rgba(0,0,0,...)` so they don't
      impose a warm cast on cool themes.
    - `.saga` and `.quest-details` card overlays switched from warm
      `rgba(255,245,210,...)` washes to neutral white-up / shadow-down
      gradients (`rgba(255,255,255,X)` over `rgba(0,0,0,X)`) — they now
      pick up whatever parchment color the active theme provides instead
      of forcing cream.
    - Each theme's palette pushed to a much bolder hue so the cue is
      unmistakable when switching characters: Soldier reads olive,
      Ranger reads sage-green, Barbarian reads tan-hide with blood-red
      accents, Rogue reads slate-gray + silver, Divine reads sapphire-on-
      ivory, Arcane reads lavender + magenta, Necrotic reads bone-green
      + bilegreen wax, Knight reads silver-blue + crimson, Scholar stays
      the original cream + red wax.
- **Per-character themes — 9 D&D archetypes (Build 05242026.18+).**
  Each character now binds to a theme palette. Switching the active
  character switches the page's color scheme; the body background
  cross-fades over 400ms.
    - **Scholar** (default) — parchment, red wax, gold seals
    - **Soldier** — tan canvas, brass, oxblood
    - **Ranger** — forest cream, copper, moss
    - **Barbarian** — hides, bone, blood-red
    - **Rogue** — slate, gunmetal, poison-red
    - **Divine** — ivory, gold, sapphire
    - **Arcane** — lavender parchment, amethyst, silver
    - **Necrotic** — bone, bilegreen, deep purple
    - **Knight** — pale silver-blue, heraldic red, polished gold

  Implementation: each theme is a CSS variable override block under
  `[data-theme="..."]` selectors. The body background is `var(--body-bg)`.
  `applyTheme(themeId)` stamps `<html data-theme="...">` and the cascade
  takes care of every accent color in one tick. Schema bumped to **v4**:
  `state.characterThemes[name] = themeId` (default `"scholar"` is seeded
  for any character missing one during `ensureShape`). The Manage panel
  gained a theme dropdown per character that re-themes the page live on
  change; the Add Character row got a theme dropdown for new characters.
  Master reset re-applies the default theme; Import re-applies the
  imported active character's theme.

  Export carries `characterThemes` like every other durable field, so
  exports remain a complete backup.
- **Removed header Reaper Remaining stat (Build 05242026.17+).** The
  `☠ Reaper ☠ Remaining` tile is gone from the progress band; the
  per-saga and per-quest REAPER badges already convey that info.
  `charSummary().sagasFtr` is still computed for the saga-list filter
  (`state.ftrOnly`) but no longer rendered as a header stat. Header now
  shows three tiles: Sagas Complete · Heroic/Epic/Legendary · Rewards Banked.
- **Empty-state pop for Add/Manage (Build 05242026.16+).** When
  `state.characters.length === 0`, the `.char-actions` row gets an
  `.empty-state` modifier that scales both buttons up to 200×44px,
  switches them to solid borders, and fills them with wax color: Add
  becomes a filled emerald-wax primary action with a gentle 2.4s pulsing
  glow (paused on hover) and cream text; Manage becomes a same-sized
  parchment-bordered companion with a wax-red dot. Once any character is
  added the row collapses back to the discrete 140×26 dashed treatment.
- **Auto-dismiss after Add Character (Build 05242026.16+).** The
  `addCharacter()` flow now flips `charManageMode = false` after a
  successful add, so the user lands directly on the saga list with the
  new character active instead of staying in the manage panel.
- **Master Reset (Build 05242026.15+).** The footer reset button now
  wipes ALL local data — every character, every saga's quest completion
  state, every reward state, every note, every puzzle override, every UI
  pref — and returns the page to a fresh-install blank slate. It clears
  both the current `sooksSagaScroll` localStorage key and the legacy
  `sooksSagaScroll.v1` key, then re-seeds via `defaultState()` and
  re-renders. Confirmation dialog explicitly recommends Export first as
  a backup path.
- **Darkened reset button (Build 05242026.15+).** Switched from the
  faded outline treatment to a wax-red gradient with cream text and a
  ⚠ glyph — reads as a deliberate destructive action and is now visible
  against the parchment background without straining.
- **Prominent build badge (Build 05242026.15+).** Footer `.build` is now
  a small parchment-card label: Cinzel 0.92rem bold uppercase with
  letter-spacing, sitting on a gold-tinted background with a matching
  border. Easy to read at a glance for triage.
- **Schema v3 — legacy-seed cleanup (Build 05242026.14+).** `SCHEMA_VERSION`
  bumped to 3. On first v2-or-earlier load, `stripUnusedLegacySeed()` runs
  once: if the character list is exactly the legacy `["Xouk","Xiin","Xokotli"]`
  trio AND none of those characters have any quest completion data
  (`charHasNoProgress()` checks quests, reward, and notes), they're wiped
  back to a blank `characters: []`. Users who actually played with that
  trio keep them; users who only inherited the implicit defaults get the
  true blank slate. Subsequent loads see `schemaVersion: 3` and skip the
  cleanup.
- **Saga header spacing (Build 05242026.14+).** The Banked / Incomplete
  pill now has `margin-left: 18px` to breathe a little further from the
  REAPER badge.
- **Last character is deletable (Build 05242026.14+).** Dropped the
  "you need at least one character" guard from `deleteCharacter()`. The
  empty-state (zero characters) is now valid, so wiping all characters
  just returns the user to the Add Character prompt. `activeChar` falls
  back to `null` when the list empties.
- **REAPER badge: main word stays "REAPER", tag flips below on completion (Build 05242026.13+).**
  Both the saga-header and per-quest badges keep `☠ REAPER ☠` as the
  main row in every state. The small tag word changes based on state and
  re-positions via flex `order`:
    - Not done (action available): tag = "First Time" ABOVE the main row.
    - Sealed (completed on Reaper): tag = "Completed" BELOW the main row,
      faded italic + the whole badge dims to 78% opacity so it reads as
      done rather than as a call-to-action.
  The SEALED main-word change introduced earlier was rolled back — the
  user wanted `REAPER / Completed`, not `SEALED / Completed`.
- **Symmetric Add/Manage buttons (Build 05242026.13+).** Both `.char-action-btn`
  instances are now fixed-size: 140px × 26px, centered text, no shrinking.
  Same font/padding/border/radius. Eye reads them as a matched pair.
- **No counter on character medallions (Build 05242026.13+).** Dropped
  the `<span class="char-stats">X/29 sagas</span>` — the same number
  lives in the progress band below and was redundant. Medallion is now
  just the character name.
- **Truly blank-by-default characters (Build 05242026.13+).** Removed
  `LEGACY_FALLBACK_CHARACTERS` entirely. v1 saves with no detectable
  character data now land at an empty list, same as fresh installs. v1
  saves WITH character keys still migrate those keys forward.
- **Export / Import moved to upper-right corner (Build 05242026.13+).**
  The two data buttons sit in a fixed-position `.data-corner` cluster
  anchored to the viewport top-right (12px / 14px from edge, z-index 50),
  on a small parchment pad so they're legible against any color of saga
  card behind them. They're always reachable regardless of scroll
  position. Removed from the footer's reset row.
- **Saga-header REAPER badge mirrors per-quest button (Build 05242026.12+).**
  Both badges now use one shared stacked structure: a small "First Time"
  / "Completed" tag word above a larger `☠ REAPER ☠` / `☠ SEALED ☠`
  main row. The base `.ftr-badge` is `display: inline-flex; flex-direction:
  column` and `.ftr-badge.tiny` just shrinks the per-line font/padding for
  quest-row use. When a quest is sealed on Reaper (or every quest in a
  saga is sealed), the tag swaps to **"Completed"** (faded `rgba(43,26,8,0.45)`
  + italic via `.ftr-badge.sealed .ftr-tag`), so the SEALED main row leads
  visually and the tag word reads as confirmation rather than call-to-action.
  The old inline-style trick on the saga-header SEALED badge is gone —
  it uses the `.sealed` class like the quest button.
- **Paired Add/Manage actions under medallions (Build 05242026.11+).**
  The two buttons now share one `.char-action-btn` class — identical
  padding (4px 14px), font (Cinzel 0.68rem), border (1px dashed
  parchment-edge), and pill shape (50%/35% radius). They sit in their
  own `.char-actions` row below the medallions instead of competing with
  them for space on the same line. A 5px colored dot in front of each
  label (emerald for Add, wax-red for Manage) is the only visual
  differentiator. `.char-select` switched to `flex-direction: column`
  with a `.char-row` wrapper for the medallions. Old `.char-add-btn` and
  `.char-manage-btn` CSS removed.
- **Reaper button stacked label + filter rename (Build 05242026.10+).**
  The per-quest Reaper badge now stacks a small Cinzel "First Time" tag
  above the larger `☠ REAPER ☠` (or `☠ SEALED ☠` when complete). The
  filter toggle dropped the skulls and now reads simply
  **First Time Reaper**.
- **User-editable puzzles (Build 05242026.10+).** Wiki-sourced puzzle
  solutions are now overridable per quest. Each puzzle entry in the
  quest details has a ✎ Edit button that opens an inline form (Type /
  Solution / Notes) with Save / Cancel / Delete. A `+ Add puzzle` button
  is always present, even on quests with no wiki puzzles. Edited puzzles
  show an "edited" badge next to the section header; a `↺ Reset to wiki`
  button discards overrides and restores the wiki default. Overrides are
  stored in `state.puzzleOverrides[questName] = [{type, solution, notes}, …]`
  (per-quest-name, not per-character — puzzle solutions are universal).
  `state.puzzleEditing` is the transient "which row is open" flag and is
  stripped from saves and exports.
- **Export / Import (Build 05242026.10+).** Two new footer buttons:
    - `⤓ Export` downloads `sooks-saga-scroll-YYYY-MM-DD.json`, a
      structured payload `{app, exportedAt, build, state}` containing
      every character, every saga's quest completion state, every
      reward state, every per-saga note, every puzzle override, and
      UI prefs.
    - `⤒ Import` reads a JSON file (accepts both the wrapped
      `{state: …}` shape and a bare state object), prompts for
      confirmation, then replaces in-memory state and re-renders.
      Imported data is run through `ensureShape()` so future schema
      migrations apply automatically.

  **All future builds must call `ensureShape()` on imported / migrated
  data and preserve fields the running build doesn't recognize** — adding
  a passthrough copy step before mutation prevents data loss if a user
  imports a JSON from a newer build into an older one.
- **Readability + layout polish (Build 05242026.9+).**
    • Loaded **EB Garamond** from Google Fonts. All bestower-name
      contexts (saga-info `.val`, `.arc-giver`, `.quest-giver`,
      `giverList` NPC names) now render in EB Garamond at ~1rem so
      proper fantasy names ("Quamak", "Saphelsa Stamiss", "Cassiopia
      Carragher") read cleanly; the decorative IM Fell English stays as
      fallback / for ambient body text.
    • **Quest-row order rearranged:** the expand toggle (`▸/▾`) moved
      to the FAR LEFT of the row, sitting before the wax-stamp
      completion-controls cluster. The toggle was restyled to share the
      cluster's parchment-pearl look (22px round, same baseline).
    • **Quest-details card redesigned** with stronger section
      definition: each `.qd-row` is now a discrete strip with a wax-red
      dotted top divider, a 100px key column in `Cinzel` uppercase with
      a wax-red color and a right-side dotted rule separating it from
      the value column, and alternating-row tint for visual rhythm.
      Story / Monsters / Traps / Puzzles / Survival now read as five
      framed sections instead of one undifferentiated block. The whole
      card got a parchment-gradient background, a wax-red left border,
      and a footer strip for the wiki citation. A media query below
      640px stacks the key above the value so labels stay readable on
      narrow screens.
- **Cross-saga sync is tier-scoped.** `SHARED_QUESTS` is keyed by
  `tier::questName` (see `sharedKey()`). Ticking a quest cascades to every
  saga of the **same tier** that lists the exact same quest name. Heroic
  and Legendary copies of the same quest name stay independent — they are
  separate accomplishments. The ↔ N indicator next to a quest reflects
  within-tier overlap only.
- **Story arcs (v.05232026.12+).** Some sagas have a multi-arc structure
  where the saga giver sends you to "story arc givers" who in turn bestow
  the individual quests. When a saga object has an optional `storyArcs`
  array (each entry `{name, giver, giverLocation, quests}`), the renderer
  groups the quest list under named arc headers with linked arc-giver
  names. Sagas without `storyArcs` fall back to the flat list. Currently
  mapped: **e-menace** (three
  arcs (The Darkening / City of Portals / The Queen of the Demonweb)
  bestowed by Adamor Thellik / Korrex Dunhaven / Xarathas respectively,
  plus a Side Quests group for Don't Drink the Water and In the Belly of
  the Beast). **h-pirates** and **e-pirates** added in `v.05232026.13` —
  the wiki only formalizes one arc per Pirates saga (Sentinels of
  Stormreach, chain bestowed by **Taggart d'Deneith** in the House
  Deneith Enclave); the Three-Barrel Cove pack quests have no chain
  giver and render as a labeled side-quest group. **h-vecna** and
  **l-vecna** added in `v.05232026.15` — wiki groups quests as Part One
  (3 quests), Part Two (6 coalition quests), and 4 side quests; no chain
  bestower NPC is named separately from the saga giver (Aster / Orm the
  Trail-blazer); per the user's in-game discovery in `v.05232026.16`,
  **Saphelsa Stamiss** chains Part One and **The Traveler** (appearing
  as a dragon) chains Part Two — both stand in Morgrave University -
  Upper Commons. h-cormyr, h-huntsilvers, and others may
  have similar structure — not yet mapped.
- **Puzzles (v.05232026.17+).** Quest detail entries can now carry an
  optional `puzzles: [{type, solution}]` array. When present, a "Puzzle"
  row renders between Traps and Survival, with each entry shown as
  `Type: solution` (solution styled in red ink for visibility). Multiple
  puzzles per quest render on separate lines. In `v.05232026.18` a full
  Vecna sweep populated **12 puzzles across 6 quests**: Bark and the
  Blade (2 — floating stones, rune wheel passcode 61953), Law and Order
  (2), Evil We Know (2), Grand Theft Aureon (4 — button sequence
  4-6-2-3-8-5-7-1, mirror, floor, beam maze), Devils to Pay (1), What
  Dreams May Come (1 — Feather + Set of Clothes for the anxiety dreams).
  Where the wiki marks a puzzle as randomized per run, the solution
  field says so explicitly instead of fabricating a fixed answer. The
  other 7 Vecna quests have no puzzles per wiki. Going forward, all new
  quest research should include puzzle solutions when the wiki or
  user-provided info has them.
- **Quest detail data crosses tiers.** `lookupQuestDetails()` falls back
  to any other saga (any tier) that has the quest, with a final
  normalized-name pass that strips (in order): `(Heroic)` / `(Epic)` /
  `(Legendary)` tier suffixes, a leading `Return to ` prefix, and a
  leading `The ` / `A ` / `An ` article (see `normalizeQuestName()`).
  Same lore, same monsters, same wiki page — sourced once, displayed
  everywhere. The `Return to ` + article strip together close the full
  Heroic ↔ Epic Gianthold gap: all 10 quests now resolve to the same
  normalized key, including the two that DDO renamed across articles
  (Heroic "The Prison of the Planes" ↔ Epic "Return to Prison of the
  Planes", Heroic "A Cabal for One" ↔ Epic "Return to Cabal for One").
  The normalization is safe — verified against every quest in the SAGAS
  array, no two distinct quests collapse to the same key. Trailing
  parens like `(Part One)` and `(monster)` are preserved (only the
  three tier suffixes are stripped).
- **Wiki-sourced data (no inference).** 19 saga stories and 51 unique
  quest detail entries pulled from DDO Wiki via Chrome MCP. Resolves to
  **126 / 266 quest slots** (47%) via the cross-tier fallback — the entire
  Heroic Saltmarsh saga and all 7 Epic sagas are fully populated. Anything
  not yet sourced renders as "Not yet sourced — see DDO Wiki ↗" with a
  direct link, never fabricated text. Per-quest givers for the 4 Sentinels
  quests in h-pirates were corrected in `v.05232026.11` — the wiki infobox
  lists Tyn/Galton/Lille/Greigur d'Deneith as bestowers (the earlier data
  had consolidated them all under Taggart d'Deneith, who is actually a
  reward NPC, not a bestower).
- **Saga rows align via CSS grid.** `.saga-header` uses
  `grid-template-columns: 20px minmax(180px,1fr) 90px 120px 130px 100px`
  so the level / Reaper / reward / quest-count badges stack into clean
  vertical columns. Two media queries tighten the columns at 820px and
  collapse to wrap-flex below 700px.
- **Path block — explicit step-by-step route for new players (Build 05242026.28+).**
  The single "Entrance" row introduced in 27 was upgraded into a full
  6-step numbered Path block, designed for players who don't know DDO's
  geography. Each quest's details card now renders:
    1. **Saga** — which saga this quest belongs to (+ level range)
    2. **Pack required** — what adventure pack to own
    3. **Talk to** — the bestower NPC + where they stand
    4. **Travel to** — the route from the bestower to the dungeon door
    5. **Dungeon door** — the specific spot
    6. **Takes place in** — the instance name
  Optional **Shortcut** callout below for teleports / exit drops.
  Pulls saga + pack from `SAGAS` (no per-quest data needed), bestower
  name from `QUEST_GIVERS`, bestower location from new `NPC_LOCATIONS`
  lookup, and travel/doorLocation/takesPlaceIn/shortcut from new
  optional fields on `QUEST_DETAILS`. Missing pieces fall back to
  "Not yet sourced — see wiki" rather than inferring.
  Backfilled: the 5 Sentinels of Stormreach quests (Bargain of Blood,
  Black Loch, Storm the Beaches, Tide Turns, Spies in the House) as
  the template. The remaining ~48 unique quest entries get the same
  treatment in subsequent builds (Pirates TBC half next, then the
  other 12 Heroic sagas, then Epic + Legendary).
  Renderer: `renderPathBlock(sagaId, questName, det)` returns the qd-row
  HTML and is called from both the unverified and verified branches of
  `renderQuestDetails()`. CSS: `.qd-path`, `.qd-path-tag`,
  `.qd-path-shortcut`. Replaces the temporary `entrance` field from
  Build 27 (kept as legacy fallback inside `renderPathBlock` when
  `takesPlaceIn` is absent).
- **Tier filter + symmetrized filter row (Build 05252026.1+).** The
  page-level filter row gained a Tier segmented control (All / Heroic /
  Epic / Legendary) so users can focus on a tier without scrolling.
  Implementation: new `state.tierFilter` ("all"|"Heroic"|"Epic"|
  "Legendary"), applied in `renderSagaList()` before grouping. Order of
  controls (left → right): Group by · Tier · First Time Reaper ·
  Expand all. Every control shares one `.ctl-input` baseline (32px
  height, Cinzel 0.82rem, same padding/border) and the four tier
  segment buttons are fixed-width (90px) so they line up symmetrically
  regardless of label length. Text labels carry meaning (color-blind
  friendly per user preference); active state is wax-red on parchment.
- **★ SKIP ★ — Shard / VIP Skip button (Build 05252026.1+).** New
  per-quest button to the right of the ☠ REAPER ☠ button. Same
  size, same tag-over-main structure, neutral brass/parchment palette
  so it doesn't compete with Reaper's wax-red. Tracks DDO's
  Astral Shard / VIP quest-skip mechanic. Labels:
    - tag (above): "Shard or VIP" — in every state, unchanged
    - main (between the stars): "SKIP" when not marked,
      "SKIPPED" when marked. ONLY the word changes on click; the tag
      stays put, no italics, no reordering.
  Quest state union extends from `false|"normal"|"reaper"` to
  `false|"normal"|"reaper"|"astral"` (schema v5; the internal value
  name stays `"astral"` even though the user-facing button reads
  "Skip" — the value name is durable, the label is presentation).
  `"astral"` counts as done for saga progress and the auto-flip to
  Banked, but does NOT count toward First-Time Reaper sealed status
  (only `"reaper"` does that). Click rules mirror the Reaper button:
    - false / "normal" / "reaper" → "astral"  (mark / promote to skip)
    - "astral" → "normal"  (downgrade; checkbox to clear entirely)
  CSS: `.astral-badge`, `.astral-badge.tiny`, `.astral-badge.sealed`,
  `.astral-badge.standby` — mirrored from `.ftr-badge` with the brass
  palette.
- **True Reincarnation Reset (Build 05252026.1+).** New button in the
  footer, sitting ABOVE Master Reset (brass/copper gradient — softer
  than Master Reset's danger-red). Resets just the active character:
  every saga's `quests` → `{}`, `reward` → `"incomplete"`, `notes` →
  `""`. Character name, theme, and other characters are preserved.
  Puzzle overrides (global per-quest) are preserved. UI prefs preserved.
  Confirm dialog explicitly names the active character being reset so
  the user can't fire it on the wrong adventurer. Both reset buttons
  share one size baseline (`.reset-btn` and `.reincarnate-btn` both
  resolve to width: 380px, same font + padding) so they read as the
  same control family — only color differs. Master Reset confirm
  dialog uses the same bullet-list pattern (everything that will be
  wiped, line by line, before the action fires).
- **Masterminds of Sharn audit + corrections (Build 05242026.29+).**
  User-reported in-game observation: Anton Westmore bestows A Sharn
  Welcome (per-quest), but **Summer Korranor** bestows the whole Part
  One story arc. Wiki re-verification confirmed:
    - **Part One chain bestower:** Summer Korranor (Dwarf, Clifftop
      Adventurer's Guild). Stands at the outer edge of Sharn -
      Clifftop Tower District's upper area, first NPC southwest from
      the airship access. Earlier data had Anton Westmore as the
      Part One chain bestower — wrong; he's a per-quest bestower for
      A Sharn Welcome only.
    - **Part Two chain bestower:** Borian Haldorak (Dwarf, Sharn City
      Councilor) in the Clifftop Tower District's lower courtyard.
      Earlier data had Celwyn Ulth — wrong; he's a per-quest bestower
      for No Refunds only.
    - **Saga npcLocation refined** to "in the Drunken Dragon tavern
      main floor near the entrance, sitting across from Bellman Rulth"
      (wiki-accurate vs. earlier "Drunken Dragon").
    - **Lost at Sea** pre-quest givenLocation refined: Issabet Tremont
      is in The Harbor of Stormreach (free-to-play). Earlier data said
      "dock just south of the Leaky Dinghy tavern" which isn't on the
      wiki — replaced with the wiki-verified text.
  Same fabrication pattern as Pirates (Build 26): per-quest bestowers
  were being mis-listed as chain bestowers. The audit pattern is now
  clear — for any saga with multi-arc structure, verify the Story Arc
  givers via dedicated wiki pages (search "Bestows Quest(s):
  <Arc Name>"), don't infer from the first per-quest NPC.
- **No-cache headers + theme color labels + footer polish (Build 05252026.2+).**
  Three small but meaningful polish items:
    - Added `<meta http-equiv="Cache-Control" content="no-store, no-cache, must-revalidate, max-age=0">`
      plus `Pragma: no-cache` and `Expires: 0` in `<head>` so the
      browser stops serving a stale copy on reload. Hitting the file
      now always reads the latest disk version.
    - Theme labels switched from archetype names ("Scholar", "Soldier",
      ...) to the **prominent color** of each theme: Carmine, Oxblood,
      Moss, Crimson, Slate, Sapphire, Amethyst, Bilegreen, Silver.
      The archetype name moved into `flavor` (tooltip) so the original
      metaphor stays discoverable on hover. `THEMES[i].id` stays
      archetype-named (durable schema key) — only the user-facing
      label changed.
    - True Reincarnation button text shortened to "↻ True
      Reincarnation" — the previous "— Reset Active Character"
      suffix caused a word wrap that made the button taller than
      Master Reset. Both buttons now sit on one line each at the
      shared 380px width.
    - Footer `.build` badge font dialed back: 0.7rem (was 0.92rem),
      weight 600 (was 700), lighter ink + lighter border. Still
      readable for triage, no longer competing with the reset
      buttons for attention.
- **Skip cap, button parity, Reaper click feel (Build 05252026.3+).**
  Three tightly linked refinements to the per-quest control cluster:
    - **Skip cap (per-saga): max 2 Skips per saga.** A new constant
      `SKIP_CAP_PER_SAGA = 2` enforces DDO's intent that Astral Shards /
      VIP cover individual quest skips, not whole-saga shortcuts. When
      a saga already has 2 quests marked `"astral"`, the remaining
      Skip buttons in that same saga render with `.disabled` and the
      native `disabled` attribute (greyed, desaturated, cursor:
      not-allowed, tooltip explains why). Quests already marked Skip
      stay clickable so the user can clear one to free a slot. Cap is
      purely UI — `"astral"` state is unaffected if loaded from import.
    - **Reaper and Skip buttons share one fixed min-width (110px).**
      Sized to fit the longest state ("SKIPPED") so clicking never
      re-flows the row. Reaper button matched to the same width so the
      pair reads as one balanced control cluster.
    - **Satisfying Reaper click + distinct sealed visual.** Sealed
      Reaper got a more dramatic blood-and-gold seal treatment
      (deep crimson core, gold rim, glow) so it no longer reads as
      "the unclicked Skip button". On click, a `.reaper-pop` class is
      attached for one render (cleared by `reaperPopQueue.delete`),
      driving a 480ms `@keyframes reaperSealPop` animation: scale up
      to 1.14× with an outward gold glow, then settle. Color-blind
      safe: REAPER + SEALED + Completed text labels carry meaning;
      animation is feedback, not state.
- **Unknown-value passthrough in ensureShape (Build 05252026.4+).**
  Hardening for schema contract Rule 4 ("Preserve unknown fields with
  a passthrough"). Previously `ensureShape` coerced any quest-state
  string that wasn't `false`/`"normal"`/`"reaper"`/`"astral"` back to
  `"normal"` — which silently downgraded "astral" → "normal" if a v5
  export was loaded into a v4 build. Worse, it would also strip any
  FUTURE value (e.g. a hypothetical "raid" added in a later build)
  before re-export. The coerce-unknown branch is now removed: known
  legacy values still migrate forward (`true` → `"normal"`, falsy →
  `false`), but any other string passes through verbatim. The UI
  already treats any truthy non-"reaper"-non-"astral" string as a
  generic "done" state, so unknown values render correctly (count
  toward saga progress + Banked auto-flip; don't count toward FTR
  sealed). Build 5+ saves are now fully forward-compatible with
  builds that add new completion modes.
- **Skip / Reaper button locked to fixed 140px footprint (Build 05252026.5+).**
  Build 3's `min-width: 110px` was a floor, not a ceiling — "★ SKIPPED ★"
  has more characters than "★ SKIP ★" and naturally exceeded the
  minimum, so the button still grew on click. Both `.ftr-badge.tiny`
  and `.astral-badge.tiny` now have `width: 140px; min-width: 140px;
  max-width: 140px` so the footprint is clamped at the larger
  "SKIPPED" size regardless of state. 140px comfortably fits the
  longest main label plus 9px padding on each side; the Reaper button
  gets the same footprint per user direction (both the larger size,
  balanced as a pair).
- **End of Eberron audit + Skip saga-locality (Build 05252026.6+).**
  Two fixes from a user-feedback audit pass:
    - **The End of Eberron (Epic) saga: 3 → 11 quests.** Wiki confirms
      the full saga is the Web of Chaos arc (Lords of Dust, Servants
      of the Overlord, Spinner of Shadows), a connecting quest
      (Beyond the Rift), the City of Portals arc (Houses of Rusted
      Blades / Broken Chains / Death Undone / The Portal Opens), and
      the Queen of the Demonweb arc (Trial by Fury, The Deal and the
      Demon, Reclaiming the Rift). `e-eberron` SAGAS entry expanded
      with the missing 8 quests, restructured into 4 storyArcs (Web
      of Chaos has no chain giver per wiki; Korrex Dunhaven for City
      of Portals; Xarathas for Queen of the Demonweb). QUEST_GIVERS
      filled for the 4 new buckets (Dectaran / Felnis / Orrex /
      Tavnar Kronnak). The 4 City-of-Portals quests share details
      with e-menace via cross-saga lookup, so QUEST_DETAILS for
      those came along for free.
    - **Skip is now saga-local.** `applyQuestState` previously
      cascaded ALL state changes (including `"astral"`) across every
      saga of the same tier that shared the quest name — meaning
      Skipping "Times Long Past" in saga A also marked it Skip in the
      half-dozen Eveningstar sagas that share it. Fixed: any
      transition involving `"astral"` (entering OR exiting) is now
      saga-local. Other transitions (checkbox, Reaper) still cascade
      across shared sagas BUT preserve any sibling saga's existing
      `"astral"` mark. The user can now Skip a quest in one saga and
      run / Reaper it normally in another without interference.
- **e-eberron arc names normalized (Build 05252026.7+).** Reverted the
  inline level/region tags I'd added to the e-eberron storyArcs
  ("The Web of Chaos (level 21, in The Harbor)", etc.) to bare names
  ("The Web of Chaos") so the entry matches the project-wide
  convention every other saga's arcs follow.
- **e-eberron QUEST_DETAILS — strict wiki source-of-truth (Build 05252026.8+).**
  Added a new `e-eberron` bucket in QUEST_DETAILS with full wiki-sourced
  entries for the 4 quests the saga adds beyond what e-menace already
  covers: The Lords of Dust, Servants of the Overlord, The Spinner of
  Shadows, and Beyond the Rift. Each entry carries story (paraphrased
  from the wiki Overview), monsters (full Monsters table with per-NPC
  wiki URLs), traps (from the Known Traps section verbatim where
  succinct), and notes (distilled from Tips and Misc — survival tips
  only, no inference). The other 7 End of Eberron quests (City of
  Portals 4 + Queen of the Demonweb 3) still resolve via cross-saga
  lookup against e-menace's existing wiki-verified entries. Net
  effect: every one of the saga's 11 quests now surfaces full quest
  details in the tool, all from wiki sources with citations.
- **Schema v6 + inline quest level + editable Path/Traps/Survival (Build 05252026.9+).**
  Three coupled changes, all additive:
    - **Schema bump 5 → 6.** New `state.questOverrides[questName] = { path?, traps?, survival? }` map for user edits. `ensureShape` defaults to `{}` on existing saves; transient editing flag `state.questOverrideEditing` is stripped by `saveState`. Migration ladder updated.
    - **Inline quest level chip.** `QUEST_DETAILS` entries can now carry an optional `levels: { heroic: N, epic: N }` field. The quest row renders a small `lvl X` chip next to the quest name, picking the value matching the saga's tier. Cross-tier shared quests show the right level for whichever saga they're being viewed in. Currently populated for the 4 e-eberron Web-of-Chaos / Beyond-the-Rift entries as the template; the rest backfill as wiki-research waves complete.
    - **Editable Path / Traps / Survival rows.** Each gets a small ✎ Edit button. Click opens an inline textarea with Save / Cancel / Reset-to-wiki controls (same pattern as the puzzle editor). When a row has an override, the wiki body is replaced wholesale and the key label gets an "edited" tag. Empty save clears the override (same as Reset) so users can't end up with a blank visible row. Path override replaces the structured numbered block with freeform notes — useful for parties with their own landmark conventions.
- **Bulk level backfill — Wave 1 (Build 05252026.10+).** Pulled the
  saga main pages from DDO Wiki for every Heroic and Epic adventure
  pack and applied `levels: { heroic, epic, legendary }` per quest,
  strictly wiki-sourced. Coverage:
    - **Saltmarsh** (10 quests, H3 / L32)
    - **Pirates: Sentinels of Stormreach chain** (4 quests, H7/E20)
    - **Pirates: Three-Barrel Cove half** (5 heroic-only at lvl 5-7)
    - **e-pirates exclusives** (5 quests, E25)
    - **Spies in the House** (H8/E21)
    - **Feywild** (13 quests, H5-6 / L32, per-quest)
    - **Isle of Dread** (12 quests, H7-8 / L33-34, per-quest)
    - **Myth Drannor** (13 quests, H13/L35)
    - **Mists of Ravenloft** (12 quests, H10-12 / L31-32, per-quest)
    - **Chill of Ravenloft** (13 quests, H8/L36)
    - **Vecna Unleashed** (13 quests, H18/L34)
    - **Masterminds of Sharn** (10 quests, H15-16 / L32 per Part)
    - **Cogs of the Mind** (8 quests, H15-16 / L32 per pair)
    - **Cormyr / Shadow / Druid / High Road / Storm Horns**
      (24 quests, varying H15-19 / E23-27, per pack)
    - **e-eberron Web of Chaos** (4 quests, H16/E21 — done in Build 9)
    - **Menace of the Underdark** (13 epic-only quests, E21-23 per arc)
    - **Gianthold** (10 quests, H13-14 / E24)
  Remaining: `e-planeswalker` quests, h/l-vecna double-check, any
  isolated stragglers. Bulk backfill used a small Python helper
  (`/tmp/apply_levels.py`) that inserts `"levels": {...}` right after
  each quest entry's opening brace, idempotently (skips entries that
  already have a level field). Cross-tier shared quests (e.g.
  Eveningstar overlap) all read the right level for whichever saga
  they're being viewed in via the existing `lookupQuestDetails`
  cross-saga fallback.
- **Path edit pre-fills the template + level chip moves after giver (Build 05252026.11+).**
  Two row-level polish items:
    - **Path edit textarea seeds the structured template.** Previously
      ✎ Edit opened a blank textarea (or showed the prior override).
      Now first-time editors get the labelled scaffolding pre-filled:
      `Saga:`, `Pack required:`, `Talk to:`, `Travel to:`, `Dungeon
      door:`, `Takes place in:`, plus the wiki-sourced `Shortcut:` line
      when present, plus a friendly trailing line inviting the user to
      append their own notes. Wiki-sourced values populate where we
      have them; missing values show `(not yet sourced — see wiki)`.
      Users append/edit instead of starting from a blank page. A new
      `buildPathTemplate(sagaId, questName, det)` helper does the
      formatting; the textarea is now 12 rows tall (was 6).
    - **Quest-row level chip rendered after the giver.** Was
      between the quest name and the shared indicator; now sits at
      the far right after the bestower. Same `.quest-level` styling.
- **True Reincarnation confirm refresh (Build 05252026.12+).** The
  confirm dialog now has an explicit **Preserved (NOT touched)**
  section that calls out three things the reset doesn't clobber:
  (1) the active character's name and theme; (2) every other
  character's data; (3) **per-quest edits — Path, Traps, Survival,
  and Puzzle overrides** — which are global (shared across
  characters), so any edit you make to those carries through every
  life and every reincarnation. Verified against the handler code:
  it only assigns `state.progress[name] = blankCharState()` and
  clears `state.questExpanded`; never touches `state.questOverrides`
  or `state.puzzleOverrides`.
- **Reliable cache-busting (Build 05252026.13+).** The no-cache `<meta>`
  tags in `<head>` are honored by browsers when serving over HTTP, but
  for `file://` URLs the browser falls back to its own modification-
  time heuristic and can hand the user a stale in-memory copy of the
  page. Two layers now ensure a fresh disk read:
    1. **Auto-reload on session start.** A short IIFE at the very top
       of the script checks a `sessionStorage` flag; if absent (first
       script execution in the browser session), it sets the flag and
       calls `location.reload()`. The reload re-fetches the file from
       disk; on the second execution the flag is set so we skip the
       reload (no infinite loop). Per-browser-session, not per tab.
    2. **Manual ⟳ Reload button in the footer.** Sits next to the
       Build badge in a new `.build-row` flex container. Click clears
       the session flag and calls `location.reload()` — gives the user
       an explicit one-click escape hatch that doubles up with the
       auto-reload script on the next load.
  `localStorage` (saved progress, character data) is untouched by
  either path. Session-cleared private/incognito windows skip the
  auto-reload silently; the manual button always works.
- **New park convention (Build 05252026.14+).** Project-specific
  override of the default garage parking rules:
    - **Filename:** kebab-case slug with build-number suffix using `_`
      as the decimal separator. Today's build lands as
      `sooks-saga-scroll-05252026_14.html`. The old
      `sooks_saga_scroll.html` filename is retired.
    - **Retention:** keep the latest 3 build files; delete the oldest
      when parking would otherwise produce a 4th. Gives the user a
      rollback window without letting the folder bloat.
    - `README.md` and `CONTEXT.md` stay unversioned and always
      reflect the latest build.
    - localStorage data carries forward across build files —
      switching to a newer build picks up where the older build left
      off because they share the same `STORAGE_KEY` and migrations
      are append-only.
- **Cross-saga level lookup fix (Build 05252026.15+).** Bug:
  `lookupQuestDetails()` returns the first matching entry across all
  buckets — but for the bunch of quests that appear in both Heroic
  and Epic sagas (Cormyr/Huntsilvers/Planeswalker shared with each
  other), some bucket entries carried only an `epic` level. When a
  Heroic-tier render hit those entries first, the level chip would
  silently not appear (no `levels.heroic`). Fixed by augmenting the
  duplicate entries with both tier levels:
    - `e-planeswalker`: 5 shared-with-Wheloon entries got
      `{heroic: 16, epic: 26}`; 4 shared-with-High-Road entries got
      `{heroic: 18, epic: 24}`
    - `e-cormyr` Storm Horns: 5 entries got `{heroic: 19, epic: 27}`
  Cross-saga lookup now returns a complete entry regardless of which
  tier the viewer is in. Side effects: Honor of the Huntsilvers
  quest-row level chips now render correctly (all 16 quests get
  their level from the cross-saga fallback); Perils of Cormyr's
  chips were already rendering from h-cormyr's local entries and are
  unaffected. **Perils of Cormyr quest count verified against wiki
  — 24 quests, not 25.** Heroic Vecna is in the data
  (`h-vecna` SAGAS entry + 13 quests with full Heroic 18 / Legendary
  34 levels per Vecna Unleashed wiki); any "missing Vecna" report
  was almost certainly a stale browser cache before Build .13's
  cache-busting fix.
- **Quest Completed button + saga progress bar — schema v7 (Build 05262026.1+).**
  Big change. Three pieces:
    - **Schema v7.** New `state.progress[char][sagaId].sagaDone =
      {questName: true}` map — independent of `quests[q]`. Migration
      seeds `sagaDone[q]=true` for any existing truthy `quests[q]` so
      progress counts carry forward.
    - **New per-quest button** between Reaper and Skip. Same 140px
      footprint as Skip. Tag stays "Saga Reward"; main word toggles
      "QUEST IN PROGRESS" → "QUEST COMPLETED". Muted parchment-green
      palette so it's color-blind-friendly and visually distinct from
      Reaper's red and Skip's brass. Clicking the button is
      **independent of Reaper / Skip** — neither is touched. When
      the button is sealed, the quest name gets a strikethrough.
      Cascades through `SHARED_QUESTS` (universal "did I run it",
      unlike Skip which is saga-local).
    - **Auto-bank now reads `sagaDone`, not `quests[q]`.** The new
      button is the sole driver of saga reward state. Reaper / Skip /
      checkbox remain as independent achievement / resource markers
      but no longer flip a saga to Banked on their own.
    - **Saga header progress bar.** The bare "X/Y quests" fraction is
      now wrapped in a fixed-width (130px) parchment-framed progress
      bar with a green fill showing the percentage of quests marked
      Completed. Same size across every saga so they line up cleanly
      in the header column.
- **Schema v8: Reaper and Skip are now fully independent — bug fix
  (Build 05262026.6).** Reported bug: clicking Skip on a Reaper'd
  quest silently un-checked the Reaper mark. Root cause: `quests[q]`
  was a single-value union, so "reaper" and "astral" were mutually
  exclusive. Fix: split Skip off into its own boolean map.
    - **New `state.progress[char][sagaId].skipDone[questName] = true`
      field.** Saga-local (no SHARED_QUESTS cascade), parallel to
      `sagaDone`. The ★ SKIP ★ button reads / writes only this field
      now and no longer touches `quests[q]`. Reaper still writes
      `quests[q] === "reaper"` as before.
    - **Migration v7 → v8** in `ensureShape`: for any saga where
      `quests[q] === "astral"`, also set `skipDone[q] = true`. The
      legacy `quests[q] === "astral"` value is left in place per the
      schema contract (preserve known field values); v8 just stops
      reading it. Going forward the saga-reset path normalizes those
      orphans to `false`.
    - **Result.** A quest can now be marked BOTH First-Time Reaper
      AND Skip simultaneously. Clicking one no longer disturbs the
      other. The reset-confirm dialog from `.4` / `.5` now lists every
      active mark when both are set.
- **Saga button renamed to "Complete for SAGA" + Reaper/Skip-set
  quests warn on un-toggle (Build 05262026.5).** Two changes:
    - **Button copy.** The middle-of-cluster button is now tag
      "Complete for" (or "COMPLETED for" when sealed) + main "SAGA".
      Same two-line structure as Reaper / Skip. Color flip still
      carries the state change (parchment → green); the word-tense
      change is the color-blind backup signal.
    - **Un-toggle warning when Reaper or Skip is set.** Clicking the
      master Quest Completed control (LEFT checkbox OR the "Complete
      for SAGA" button) to UN-check the green seal — while the quest
      is in Reaper or Skip state — now opens a confirm dialog. Accept
      = full row reset (clear Reaper / Skip, clear `sagaDone`,
      cascade). Cancel = nothing changes. Matches the Reaper-re-click
      behavior from `.4` so all three "I clicked by mistake" paths
      converge on the same explicit reset flow.
- **LEFT checkbox is 2-state-only + Reaper re-click confirms full row
  reset (Build 05262026.4).** Two coupled refinements:
    - **Master checkbox shows only green-or-empty.** The old "red+skull
      when reaper-done" variant was removed — it conflated Reaper state
      and Quest-Completed state on a single indicator. Now the LEFT is
      unambiguous: green seal = done, empty parchment = not done.
      Reaper state lives entirely on the Reaper button. Visual was
      cranked further (32px, 2.5px border, double-ring glow with gold
      seal-ring + outer emerald halo, 26px chunky checkmark).
    - **Reaper re-click → confirm dialog → full row reset.** Clicking
      a sealed Reaper button now opens a confirm: "Clear all marks on
      [quest]? Resets the entire row — removes Reaper, unchecks Quest
      Completed, clears Skip." Accept = full row reset (`sagaDone`
      cleared, `quests[q]` set to false, cascades through same-tier
      siblings). Cancel = no change. The old "downgrade reaper → normal"
      path was removed — users who re-clicked Reaper almost always
      wanted a full mistake-recovery reset, not a downgrade.
- **LEFT checkbox is the master Quest Completed indicator
  (Build 05262026.3).** Refactored the per-quest row so the LEFT
  control is the dramatic "Quest Completed" master:
    - **Drives `sagaDone`, not legacy `quests[q]`.** The checkbox was
      previously tied to `quests[q] === "normal"`, which fed nothing
      visible. Now it reads/writes `sagaDone[q]` — the same field the
      saga progress bar + reward auto-bank read. Bigger (28px), 2px
      border, golden glow ring, larger checkmark — meant to read at a
      glance as the master state.
    - **Reaper, Skip, and Saga Reward all auto-fill it.** Clicking any
      of the three sets `sagaDone[q] = true` as a side effect (the LEFT
      checkbox ticks itself). Clearing Reaper or Skip does NOT clear
      the checkbox — the master only flips off via the checkbox itself,
      the Saga Reward button, or saga reset.
    - **Saga reset (BANKED → INCOMPLETE) now clears Skip too.** Per
      user direction, Skip is now treated like sagaDone for saga-reset
      purposes (freeing the 2-skip cap for the next pass). Only Reaper
      survives a saga reset — that's the per-life achievement that
      only clears on True Reincarnation. Confirm dialog updated.
- **Clickable header reward pill + BANKED → INCOMPLETE saga reset
  (Build 05262026.2).** Two coupled changes:
    - **Header pill is now a button.** Previously the BANKED / INCOMPLETE
      tag on the saga-header row was display-only; the only way to flip
      it was to expand the saga and click the body button. The pill is
      now a real `<button>` with the same `data-reward` contract — click
      it directly from the collapsed row. Hover brightens it, focus
      shows a gold outline (keyboard-friendly). Body and header share
      one handler so they always reflect the same state, including
      sagas inside multi-saga containers.
    - **BANKED → INCOMPLETE wipes `sagaDone`.** When the reward toggles
      from Banked back to Incomplete, the handler clears `sagaDone` for
      every quest in that saga (and any same-tier sibling saga via
      `SHARED_QUESTS`), so the progress bar returns to 0 / N. Originally
      shipped preserving both Reaper AND Skip marks; Build 05262026.3
      changed Skip to also clear on saga reset (Reaper alone is the
      per-life achievement that survives). A `confirm()` dialog runs
      first so an accidental click doesn't erase a session of work.
      Going INCOMPLETE → BANKED has no side effects (manual bank flag).
- **Two new tiers — Non-Saga Epic + Non-Saga Legendary (Build 05272026.1+).**
  Schema bumped to **v9**. Two new tier strings are now valid:
  `"Non-Saga Epic"` (Epic-level-20 standalone raids — for leveling)
  and `"Non-Saga Legendary"` (Legendary standalone quests — for First
  Time Reaper). Group order is bookended — Non-Saga Epic first, then
  the original Heroic / Epic / Legendary, then Non-Saga Legendary
  last. Tier filter row gains two new buttons with extra width so
  the labels fit ("Non-Saga Epic" 130px, "Non-Saga Legendary" 156px;
  the original four are 80px). 29 new containers (3 Epic + 26
  Legendary) carry a new `isNotSaga: true` flag — the saga renderer
  detects it and swaps the "Saga Giver / Location" rows for a single
  italic "Not a saga" meta-note so the user never mis-reads the
  container as a real saga. Pack, Patron, and storyArc bestowers
  (where present) still render normally. Total quest count grew
  from 353 to 470. Coverage:
    - **Non-Saga Epic L20 (3 raids):** The Lord of Blades (Kenneth
      d'Cannith, House Cannith), Tower of Despair (Uloth, Amrath),
      The Dreaming Dark (Venge, Isle of Forgotten Dreams).
    - **Non-Saga Legendary (26 containers, ~104 quests):** Against
      the Slave Lords (Cerra the Gatekeeper), Attack on Stormreach
      (Darmon Kosh), Devil Assault — Legendary Chronoscope raid,
      Disciples of Rage (Haden d'Jorasco), Dragonblood Prophecy,
      Fall of the Night Brigade (Old Tarlow), Free-to-Play singles
      bucket, Grip of the Hidden Hand (Una Noldrun), Hunter and
      Hunted, Illithid Invasion (Omryn Cedarbrook), Masterminds of
      Sharn outliers (Too Hot to Handle + Project Nemesis), Peril
      of the Planar Eyes (Burwick Borley), Secrets of the Artificers
      Legendary raids (Kenneth d'Cannith), Shadow Under Thunderholme
      raids, Slice of Life, Tavern Tales Legendary, The Devil's
      Gambit, The Dragons' Hand (Tailara Fiercewing), The Lost
      Gatekeepers (Preceptor Kroth), The Mines of Tethyamar (Asta
      the Deep-Minded), The Necropolis Part 4 — The Mark of Death,
      The Soul Splitter, The Temple of Elemental Evil (two arcs —
      Elmo / Canon Terjon + 7 per-quest bestowers), The Vale of
      Twilight Legendary, Trials of the Archons, White Plume
      Mountain Legendary.
  Source-of-truth: DDO Wiki pack pages (no inference). Pack-level
  chain bestowers populated where the wiki pack page surfaced them;
  ToEE got full per-quest bestowers because its pack page documents
  all seven. Everything else falls back to "Not yet sourced — see
  DDO Wiki ↗" — Wave 2+ will backfill per-quest bestowers, monsters,
  traps, puzzles from individual quest wiki pages, same playbook as
  the existing data. Migration v8 → v9 is **purely additive** — no
  data fields changed. Older builds that load a v9 export reset
  state.tierFilter to "all" via the existing VALID_TIER_FILTERS
  guard if they don't recognize a new value; user progress is
  otherwise intact (unknown-string passthrough rule from Build
  05252026.4 still applies to quests[q]).
- **Non-saga tier corrections — Build 05272026.2.** Five fixes after
  in-game review of Build .1:
    1. **Tier order changed.** Group order is now **Heroic → Non-Saga
       Epic → Epic → Legendary → Non-Saga Legendary** (was bookended
       in Build .1). Non-Saga Epic sits where it makes sense as a
       leveling stop after Heroic. Tier filter row reordered to match;
       `VALID_TIER_FILTERS` reordered for consistency.
    2. **Non-Saga Epic raids dropped.** Build .1 had The Lord of
       Blades, Tower of Despair, and The Dreaming Dark — all three
       are raids per the DDO Wiki Raids category page. The non-saga
       tracker does not track raids. Replaced with **Phiarlan
       Carnival** (the only non-raid, non-saga, non-pirate Epic L20
       content on the wiki) and **Time Flies** (Anniversary
       Celebration L20 quest, free-to-play). Full per-quest details
       (story, monsters via `table.monster-table`, traps, tips) are
       in QUEST_DETAILS for all 5 quests — Wave-1-complete coverage.
    3. **Non-Saga Legendary raids pruned.** The wiki Raids page
       (https://ddowiki.com/page/Raids) authoritatively classifies
       these as raids — all removed: The Chronoscope (Legendary),
       Riding the Storm Out, Killing Time, Hunt or Be Hunted,
       Legendary Master Artificer, Legendary Lord of Blades, Fire on
       Thunder Peak, Temple of the Deathwyrm, The Codex and the
       Shroud, Legendary Vision of Destruction, Defiler of the Just,
       The Mark of Death, Too Hot to Handle, Project Nemesis,
       Legendary Tempest's Spine, Legendary Hound of Xoriat, Den of
       Vipers. Five containers became empty after raid removal and
       were deleted entirely: `ns-l-devil-assault`,
       `ns-l-artificers-extras`, `ns-l-thunderholme`,
       `ns-l-necropolis-4`, `ns-l-masterminds-extras`.
    4. **Saga-overlap audit clean.** Cross-checked all 91 non-saga
       quest names against every existing saga's `quests[]` array via
       an integrity script — 0 overlaps after also dropping Lost at
       Sea (which lives in `l-masterminds`'s pre-quest arc) and
       moving Time Flies into the Non-Saga Epic bucket.
    5. **Final tally:** 23 non-saga containers (2 Epic + 21 Legendary)
       carrying 91 quests (5 Epic L20 + 86 Legendary). Total project
       quest count: 454. Schema v9, build 05272026.2.
  Phiarlan Carnival arc giver is **Rouge** in the House Phiarlan
  Enclave near the Marketplace Gate to the Southeast; per-quest
  bestowers are Glitter Underhill (A Small Problem), Cyan
  (Partycrashers), Ochre (The Snitch), Cerise (Under the Big Top).
  Time Flies is bestowed by Sariyon Dran-dal at The Anniversary
  Celebration. Per-quest QUEST_DETAILS (Wave 2 backfill for the 86
  Legendary quests) is the next research pass — Build .1's structure
  + bestowers stays, only monsters/traps/tips need individual quest
  page fetches.
- **Wave 2.1 — per-quest level reference for all 86 non-saga Legendary
  quests (Build 05272026.3+).** Matching the saga-quest schema, every
  non-saga Legendary quest now carries `QUEST_DETAILS[containerId]
  [questName].levels = {legendary, heroic}` sourced verbatim from the
  DDO wiki Quests-by-level-and-XP Legendary table (format e.g. "31
  (H8)" → `{legendary: 31, heroic: 8}`). `wikiUrl` populated per quest.
  Entries flagged `monstersUnsourced: true` are stubs awaiting Wave 2.2
  per-quest wiki page visits (story / monster table / Known Traps /
  Tips and Misc). Wave 2.1 is the "level reference" deliverable; Wave
  2.2 will swap each stub's story line + add monsters / traps / tips
  per quest. All 21 non-saga Legendary containers covered (86 quests
  total). Coverage breakdown:
    - Slave Lords (3 quests, L31/H8 — chain order required first pass)
    - Attack on Stormreach (4, L34/H13)
    - Disciples of Rage (5, L31/H14)
    - Dragonblood Prophecy (2, L31/H10 — raids dropped)
    - Fall of the Night Brigade (4, L34–35/H4)
    - Free-to-Play singles (10, L30–36)
    - Grip of the Hidden Hand (5, L33/H16)
    - Hunter and Hunted (2, L32/H7 — raid Hunt-or-Be-Hunted dropped)
    - Illithid Invasion (4, L34 — no heroic version)
    - Peril of the Planar Eyes (4, L32/H12)
    - Slice of Life (3, L34/H18)
    - Tavern Tales Legendary (2, L35)
    - The Devil's Gambit (4, L30/H14)
    - The Dragons' Hand (5, L36/H10)
    - The Lost Gatekeepers (5, L32/H3)
    - The Mines of Tethyamar (5, L31/H15)
    - The Soul Splitter (6, L32/H17)
    - ToEE (7, L34/H8)
    - The Vale of Twilight (2, L31/H15 — raids dropped)
    - Trials of the Archons (3, L30/H13 — raid Defiler dropped)
    - White Plume Mountain (1, L32/H9)
- **Non-saga UX cleanup (Build 05272026.4+).** Two surgical changes
  targeting `saga.isNotSaga === true` containers only — saga rows are
  unaffected:
    1. **Skip button hidden.** Per-quest ★ SKIP ★ / ★ SKIPPED ★ button
       does not render for non-saga rows. The `SKIP_CAP_PER_SAGA`
       cap mechanic, the `skipDone` storage field, and the
       click-handler binding all remain in place so saga rows keep
       behaving exactly as before and any v8/v9 export with skipDone
       fields round-trips losslessly.
    2. **Banked → Completed.** Non-saga containers don't have a saga
       reward, so "Banked"/"Incomplete" labels become "Completed"/
       "Incomplete". The on-state CSS class `.banked` stays as the
       visual hook (green wax) — the value name is durable, only the
       presentation flips. Touched: the header reward pill, the body
       reward toggle button, the "End Reward" label (now "Tracking"
       for non-saga), and the BANKED → INCOMPLETE confirm dialog
       (drops the Skip-slots bullet, says "container" instead of
       "saga"). Per-quest "Complete for SAGA" button becomes plain
       "Mark as COMPLETE" / "Marked COMPLETE" on non-saga rows so the
       wording stops referring to a non-existent saga. Saga progress
       bar tooltip drops "for saga reward" suffix for non-saga.
  Schema is unchanged — `reward: "banked"` / `"incomplete"` still
  the only two valid values. v8 → v9.1 migration is purely
  presentational.
- **Wave 2.2 — comprehensive per-quest backfill for all 86 non-saga
  Legendary quests (Build 05272026.5+).** Every entry in
  `QUEST_DETAILS["ns-l-*"]` now carries the same shape as the existing
  saga-quest entries — `levels` (from Wave 2.1, untouched), `story`
  (verbatim from each quest's wiki Overview section), `monsters` (full
  list from `table.monster-table` with per-monster wiki URLs), `traps`
  (from Known Traps section, or `trapsNone: true` where the wiki
  doesn't surface a traps section), `notes` (distilled from Tips and
  Misc), and `wikiUrl`. Fetched via Chrome MCP — no inference, all
  text comes verbatim from the wiki. Integrity check: 86 quests, 86
  with monsters (100% coverage). Story/monsters/traps/tips render in
  the quest-details card alongside the level chip and wiki link, same
  as saga quests. Total story+monster+traps+tips data added: ~210kb
  to the build file.
- **Level-chip fix for non-saga rows (Build 05272026.6+).** The
  level-chip lookup used `tier.toLowerCase()` to index `levels` — fine
  for the original three tiers but it produced `"non-saga legendary"`
  / `"non-saga epic"` for the new tiers, neither of which is a key in
  any `QUEST_DETAILS.levels` map. As a result the level chip silently
  didn't render on non-saga quest rows even though the data was
  there. Fix: explicit mapping — `"Non-Saga Epic" → "epic"` and
  `"Non-Saga Legendary" → "legendary"` — so the Wave 2.1 levels data
  flows through to the chip on every non-saga row. Saga rows
  unaffected. Schema unchanged.
- **Tier filter restyle + FTR toggle removed (Build 05272026.7+).**
  Three coupled controls-row changes:
    1. **Chevron stack.** The tier buttons (Heroic, Non-Saga Epic,
       Epic, Legendary, Non-Saga Legendary) are now right-pointing
       chevron pentagons via CSS `clip-path`, overlapping
       breadcrumb-style. Reads as a leveling progression rather than
       a flat segmented control.
    2. **All on the far right.** "All" is split off from the chevron
       stack by a 22px spacer and styled as a rounded reset pill so it
       sets visually apart from the progression.
    3. **First Time Reaper toggle removed.** The per-saga REAPER badge
       and the per-quest Reaper button already convey FTR state per
       row; the filter was redundant. `state.ftrOnly` stays in the
       schema for v8/v9 export compatibility but is forced to false on
       first load if a save had it set, so no orphaned filter can
       silently hide content.
- **Wave 2.2.1 — DOM-based re-fetch of 6 quest stragglers (Build
  05272026.7+).** Audit flagged 6 of the 86 non-saga Legendary quests
  with thin/missing tips or story after Wave 2.2 (the flat-text regex
  extractor over-matched footer navigation or under-matched empty
  Overview sections). Switched to a DOM-based section extractor that
  reads each `<h2>`'s siblings until the next `<h2>` — clean section
  boundaries, no false positives. Quests fixed: Secret of the Slavers'
  Stockade, Dread Sea Scrolls, Beautiful Nightmares, Grim and Barett,
  Thrall of Duty (story pulled from the page intro since Overview was
  empty on that page), Desire in the Dark. All 86 now have proper
  story / tips / monsters / traps.
- **Tier filter — flat buttons + decorative arrows (Build 05272026.8+).**
  Build .7's overlapping chevron clip-path was clipping label points
  where buttons overlapped. Replaced with flat rectangular tier
  buttons (Heroic, Non-Saga Epic, Epic, Legendary, Non-Saga Legendary)
  separated by decorative double-chevron `❯❯` glyphs styled in
  wax-red, lightly translucent. The arrows do the directional work
  (left-to-right flow), buttons stay fully readable. Hover gives a
  tiny rightward `translateX(2px)` nudge to reinforce the flow
  direction. A staggered `flowPulse` keyframe ripples the arrows
  from left to right on a 2.4s loop. "All" stays on the far right
  separated by a 22px spacer, styled as a rounded reset pill.
- **Wave 3 — puzzles backfill for non-saga Legendary quests (Build
  05272026.9+).** Audited all 86 non-saga Legendary quest tips for
  puzzle indicators (`puzzle`, `riddle`, `simon`, `tile`, `rune`,
  `combination`, `mirror`, `symbol`, `sphinx`), then drilled into each
  flagged quest's DDO Wiki page via Chrome MCP to extract structured
  puzzle data. **26 puzzle entries across 20 quests in 15 containers**
  — all sourced verbatim from each quest's wiki Tips/Walkthrough
  section. Highlights:
    - **Seizing the Dawn** — full 5-layout button-sequence table
      (Layout 1: 3,4,4,1,2 ... Layout 5: 4,3,2,1) for the orb-row
      puzzle.
    - **Burning Down the House** — two wheel-puzzle solutions: Games
      Room Two (Yellow→White Triangle, Purple→Red Star) and Platinum
      Wing secret door (Red→Blue Diamond, Blue→Red Star).
    - **Multitude of Menace** — 4 runes lit facing the hallway opens
      the blue-crystal door.
    - **ToEE: Air Node** — 3 runes (two behind secret doors, one on
      statue belly via Search) + small puzzle for Dragon's Hoard.
    - **The Archons' Trial** — Test of Intelligence's trap-puzzle
      warning (the fake 4th 'absurdly easy' puzzle electrifies the
      floor — use the tiny button on the left of the exit gateway
      instead).
    - **Members Only** — kitchen-lookup rune combination (27 possible
      combos).
    - **White Plume Mountain** — Gynosphinx riddle (randomized 1 of 3).
  Schema: `puzzles: [{ type, solution, notes }]` per quest entry —
  matches the existing saga-quest puzzles shape, so the renderer
  surfaces them in the quest details card without any UI change.
- **Wave 4 — dungeon door + Path-block trim (Build 05272026.10+).**
  Two coupled changes:
    1. **`doorLocation` per quest.** Fetched the wiki "Quest acquired
       in:" infobox field for every one of the 86 non-saga Legendary
       quests via Chrome MCP. That field is the wiki's standardized
       answer for "where does this quest's entrance live", populated
       across all DDO quest pages — so it's reliable, structured, and
       wiki-sourced. 86/86 quests now carry a `doorLocation` string.
       Examples: Slave Pits chain → "The Gatekeepers' Grove"; Attack
       on Stormreach chain → "Lordsmarch Palace"; ToEE 7 quests →
       "Temple Grounds"; The Mines of Tethyamar 5 quests →
       "Tethyamar Mining Outpost"; The Archons' Trial / Demon Assault
       / The Devil's Details → "Amrath".
    2. **Path block trimmed to 4 rows.** Dropped **Travel to** and
       **Takes place in** rows from `renderPathBlock` and from the
       Path-edit template (`buildPathTemplate`). The fields `travel`
       and `takesPlaceIn` stay in the durable schema (existing saga
       entries use them) — only the renderer drops them. Path now
       shows: **Saga · Pack required · Talk to · Dungeon door**.
- **CONTEXT.md refresh (Build 05272026.10+).** CONTEXT.md was from
  May 26 — predated Wave 1/2/3, schema v9, the non-saga tiers, the
  tier-filter restyle, the level-chip + door additions. Fully
  rewritten as a slim quick-reference matching current state.
  README.md remains the full history; CONTEXT.md is the brief that
  loads fast for new conversations.
- **Wave 4 correction — quest giver location vs dungeon door (Build
  05272026.11+).** Build .10 conflated two things. The wiki "Quest
  acquired in:" infobox field is the **NPC's public area** (where the
  quest giver stands), NOT necessarily the dungeon door — in DDO some
  bestowers stand right at the entrance, others stand in a tavern or
  across a different building. Honest distinction restored:
    - **"Talk to" row** = quest giver location, contextually given
      via `QUEST_GIVERS` + `NPC_LOCATIONS`.
    - **"Dungeon door" row** = ONLY populated when a wiki source
      explicitly documents the door — no inference. Build .10's 86
      doorLocation values (which were derived from "Quest acquired
      in:") have been cleared. Until a future research wave sources
      door locations explicitly from each quest's wiki Overview text,
      non-saga Legendary entries show "Not yet sourced — see wiki"
      for the Dungeon door row.
- **Survival field formatting (Build 05272026.11+).** Was a wall of
  text. New `formatSurvivalList()` helper splits the notes string at
  sentence boundaries (`. ` + capital letter) and renders as a
  bulleted list (`<ul class="qd-survival-list">`) with parchment-fleur
  bullets (`❦` glyph in wax-red), dotted dividers between bullets, and
  EB Garamond body text at 1.55 line-height. Short notes (under 120
  chars) stay as a single paragraph since they don't need splitting.
  Falls back gracefully if there's no clear sentence structure.
- **Puzzle font + layout (Build 05272026.11+).** Restructured from
  inline `<span>` tags to a block layout with a wax-red left rule:
  Type label (Cinzel 0.78rem uppercase, faded ink), then Solution
  bumped to 1.18rem EB Garamond bold in wax-red, then Notes in italic
  EB Garamond at 1rem. Puzzle blocks now have parchment background +
  12px gap between multiple puzzles so a quest with 2-3 puzzles
  doesn't read as a wall.
- **NPC location backfill — Wave 5 (Build 05282026.1).** Built out
  `NPC_LOCATIONS` from 91 entries to **217** by sourcing every NPC
  named in `QUEST_GIVERS`, `SAGAS[].npc`, and
  `SAGAS[].storyArcs[].giver`. 49 were lifted from saga context
  (`saga.npcLocation` + `storyArcs.giverLocation` strings already
  embedded but never duplicated to the dictionary); the remaining 126
  were fetched directly from the ddowiki.com NPC page's `Location:`
  infobox via Chrome MCP. Remaining gaps (~18) are NPCs whose wiki
  page either does not exist or lacks a `Location:` field — those
  fall back to the renderer's "location not yet sourced — see wiki"
  placeholder, which is honest and lets the user fill in via the
  override system. Source rule: every entry is a verbatim quote from
  the wiki Location infobox; ddowiki's idiosyncratic spelling is
  preserved (no normalization).
- **Add/Manage button dots removed (Build 05282026.2).** The small
  accent dot rendered by `.char-action-btn::before` (and its
  `.add` / `.manage` / empty-state variants) has been stripped out.
  Buttons now show their text label only — cleaner alignment, less
  visual noise next to the medallions. Compact + empty-state
  variants both updated.
- **Mists of Ravenloft quest order corrected (Build 05282026.3).**
  Reordered both `h-mist` and `l-mists` to match the canonical wiki
  saga page order. The "In the Shadow of the Castle" arc now leads
  with **Death House** (was Fresh-baked Dreams), and "The Light of
  the Land" arc now leads with **Ravens' Bane** (was last in the arc;
  the other three quests follow). Audit notes: only the two Mists
  saga pages publish a flat ordered quest list on the wiki — every
  other saga page defers to its pack page, and the build already
  mirrors pack-page arc order, so no other corrections were needed.
- **Versioning.** Footer prefix is `Build ` (not `v.`); keep it for
  consistency. Currently `Build 05282026.3`.
- **Wiki access for future research.** Direct `mcp__workspace__web_fetch`
  against `ddowiki.com` returns empty bodies (proxy block). Use Chrome MCP
  (`list_connected_browsers` → `select_browser` → `tabs_create_mcp` →
  `navigate` → `get_page_text`) and close the tab when done.

## Data Schema & Import/Export Contract

**This contract is binding for every future build.** Import/Export is the
user's only durable backup path; any change that silently drops a field,
renames a key, or rejects a previously-valid payload is a regression.

### Current shape (schema v14, Build 07142026.1+)

Stored under `localStorage["sooksSagaScroll"]` and emitted by Export
under `payload.state`. Every field here MUST survive a round-trip
through Export → Import unchanged:

```text
state = {
  schemaVersion: 14,              // integer; bumped when shape changes
  activeChar: string | null,      // currently-selected character name
  characters: [string, ...],      // ordered roster (display order)
  characterThemes: {              // v4+: per-character theme binding
    "<charName>": "scholar" | "soldier" | "ranger" | "barbarian" |
                  "rogue" | "divine" | "arcane" | "necrotic" | "knight"
  },
  characterServers: {             // v11+: per-character DDO server for the
    "<charName>": "" | "Argonnessen" | "Cannith" | "Ghallanda" | "Khyber" |
                  "Orien" | "Sarlona" | "Thelanis" | "Wayfinder" | "Cormyr" |
                  "Moonsea" | "Shadowdale" | "Thrane" | "Hardcore"
    // "" / absent = no live DDO Audit lookup ("Missing server" on the medallion)
  },
  characterLevels: {              // v12+: per-character last-known aggregate
    "<charName>": 30              // (total) level, captured from DDO Audit and
                                  // persisted for offline level-based features.
                                  // absent until a lookup has resolved once.
  },
  characterAudio: {               // v13+: per-character Audio Alerts toggles
    "<charName>": { low: bool, mid: bool, high: bool, raid: bool }
                                  // absent = inherit the legacy global META
                                  // value (audioCues), then the defaults
                                  // (high + raid ON, low + mid OFF). Written
                                  // on first divergence, seeded from the
                                  // resolved prefs at that moment.
  },
  characterAutoComplete: {        // v14+: per-character Quest Autocomplete
    "<charName>": { on: bool, mode: "normal" | "reaper" }
                                  // absent = OFF + "normal" (nothing
                                  // auto-marks until the user opts in).
                                  // Written on first control change while
                                  // that character is active.
  },
  progress: {
    "<charName>": {
      "<sagaId>": {
        quests:   { "<questName>": false | "normal" | "reaper" | "astral" }, // v5+: +astral
        sagaDone: { "<questName>": true },   // v7+: quest-completed-for-reward (master indicator)
        skipDone: { "<questName>": true },   // v8+: Skip mark, saga-local (no SHARED_QUESTS cascade)
        reward:   "incomplete" | "banked",   // saga + non-saga both use these values
        notes:    string
      }
    }
  },
  puzzleOverrides: {              // global per quest (NOT per character)
    "<questName>": [
      { type: string, solution: string, notes: string },
      ...
    ]
  },
  questOverrides: {              // v6+: per-quest user edits replacing rendered detail sections
    "<questName>": { path?: string, traps?: string, survival?: string }
  },
  groupBy:    "tier" | "region" | "pack",
  tierFilter: "all" | "Heroic" | "Non-Saga Epic" | "Epic" | "Legendary" | "Non-Saga Legendary", // v5+, expanded v9
  ftrOnly:    boolean,           // First-Time Reaper filter (v9.3: UI toggle removed, force-cleared on load)
  expandAll:  boolean,
  questExpanded: {                // saga.id -> { questName: true }
    "<sagaId>": { "<questName>": true }
  }
}
```

Transient fields stripped before save/export (do NOT persist):
- `state.puzzleEditing` — which puzzle row is mid-edit. UI-only.
- `state.questOverrideEditing` — which quest override is mid-edit (v6+). UI-only.

### Migration ladder (always additive)

Migrations live in `migrate()` and `ensureShape()` in the latest
`sooks-saga-scroll-<MMDDYYYY>-<N>.html`. The ladder so far:

| From | To | What changed |
|---|---|---|
| v1 (unversioned) | v2 | Top-level char keys → `state.progress[name]`; added `schemaVersion`, `characters`, `questExpanded`. |
| v2 | v2 (reward migrate) | `reward: "not_earned" → "incomplete"`, `"redeemed" → "banked"` in `ensureShape`. |
| v2 | v2 (quest migrate) | `quests[q]: true → "normal"` (3-state model). |
| v2 | v3 | One-shot `stripUnusedLegacySeed()` removes the implicit `["Xouk","Xiin","Xokotli"]` default trio if none have any progress. |
| v3 | v4 | Added `state.characterThemes`; missing themes default to `"scholar"` per character via `ensureShape`. |
| v4 | v5 | Added `state.tierFilter` ("all"\|"Heroic"\|"Epic"\|"Legendary"); default `"all"`. Extended `quests[q]` value union from `false\|"normal"\|"reaper"` to `false\|"normal"\|"reaper"\|"astral"` — `"astral"` = VIP / Astral Shard skip; counts as done for saga progress but NOT toward First-Time Reaper sealed status. `ensureShape` accepts `"astral"`; unknown strings still fall back to `"normal"`. |
| v5 | v6 | Added `state.questOverrides` — a per-quest-name map of user edits that REPLACE the rendered Path / Traps / Survival sections of the quest details card when set. Shape: `{ "<questName>": { path?: string, traps?: string, survival?: string } }`. All three fields optional; missing fields fall back to wiki-sourced defaults in `QUEST_DETAILS`. Purely additive — `ensureShape` defaults to `{}` on existing saves. Editing flag `state.questOverrideEditing` is transient (stripped by `saveState`). |
| v6 | v7 | Added `state.progress[char][sagaId].sagaDone = {questName: true}` — per-quest "completed for saga reward this life" map. The new Quest Completed button is the only thing that toggles it; saga auto-bank-to-Banked now reads sagaDone counts (not quests[q] counts). Migration: any existing truthy `quests[q]` seeds `sagaDone[q]=true` on first load so users don't lose their saga-progress count when v6→v7 runs. Reaper, Skip, and the checkbox no longer flip saga state on their own — they're independent achievement/resource markers. |
| v7 | v8 | Split Skip off from `quests[q]` into a new `state.progress[char][sagaId].skipDone[questName] = true` boolean map (saga-local, no SHARED_QUESTS cascade), so Reaper and Skip can both be set on the same quest. Migration: any legacy `quests[q] === "astral"` is mirrored to `skipDone[q] = true` on first load; the legacy value is left in place per the schema contract (preserve known field values). |
| v8 | v9 | Two new tier strings — `"Non-Saga Epic"` and `"Non-Saga Legendary"` — for standalone quests not in any saga. `VALID_TIER_FILTERS` expands to accept them. `SAGAS` adds 29 containers with a new `isNotSaga: true` flag; the renderer suppresses the "Saga Giver" / "Location" rows for those and shows a "Not a saga" italic note instead. Group order bookends the existing tiers (Non-Saga Epic → Heroic → Epic → Legendary → Non-Saga Legendary). Purely additive — no existing data is touched. Older builds that load a v9 export reset `state.tierFilter` to `"all"` via the existing guard if they don't recognize the new value; user progress and quest completion data are unaffected. |
| v9 | v10 | New sixth tier string `"Non-Saga Heroic"` (the five per-pack Harbor containers). `VALID_TIER_FILTERS` accepts it. Additive tier value only — no storage-shape change; older builds reset an unrecognized `tierFilter` to `"all"`. |
| v10 | v11 | Added `state.characterServers` — a per-character map `{ "<charName>": "<DDO server>" }` for the optional DDO Audit live-status lookup. `ensureShape` defaults it to `{}`; `migrate()` v2+ path simply stamps `schemaVersion` forward. Mirrors `characterThemes` (synced on add/rename/delete). Purely additive — no existing data touched; an empty/absent server just means "no live lookup for this character." Round-trips via the whole-state Export. |
| v11 | v12 | Added `state.characterLevels` — a per-character map `{ "<charName>": <aggregate level int> }`, captured from the DDO Audit lookup (online, or last-known from the DB while offline) and persisted so future level-specific features work offline. `ensureShape` defaults it to `{}`; synced on add/rename/delete; written only when the value changes. Purely additive — no existing data touched. Round-trips via the whole-state Export. |
| v12 | v13 | Added `state.characterAudio` — per-character Audio Alerts toggles `{ "<charName>": { low, mid, high, raid } }` (Build 07122026.1; the prefs previously lived as ONE GLOBAL set in the META key `audioCues`). `ensureShape` defaults it to `{}`; a character WITHOUT an entry falls back per-field to the legacy global META value (kept in place as the seed for new characters), then to the defaults (High Reaper + Raids ON, Low + Mid OFF) — so migrated saves behave identically until a checkbox is touched while that character is active. The entry is written on first divergence (seeded from the resolved prefs so the other toggles keep their inherited values); synced on add/rename/delete like `characterThemes`/`characterServers`. Purely additive — no existing data touched. Round-trips via the whole-state Export. |
| v13 | v14 | Added `state.characterAutoComplete` — per-character Quest Autocomplete setting `{ "<charName>": { on: bool, mode: "normal"\|"reaper" } }` (Build 07142026.1). When `on`, the 15s live poll marks the quest the active character is standing in (location_id → `getQuestAreaMap()`) as complete: `"normal"` = green Quest Completed seal only (`sagaDone` cascade + auto-bank); `"reaper"` = also First-Time Reaper (`quests[q]="reaper"`, ☠ button cascade; `NO_REAPER` quests fall back to normal). `ensureShape` defaults it to `{}`; a character WITHOUT an entry resolves to OFF + `"normal"` — purely additive, no existing data touched, nothing fires until the user opts in. Synced on add/rename/delete like `characterAudio`. Round-trips via the whole-state Export. |

### Export payload wrapper (Build 05242026.10+)

Export wraps `state` so the file is self-identifying:

```json
{
  "app": "Sooks Saga Scroll",
  "exportedAt": "<ISO 8601 timestamp>",
  "build": "Build 05242026.X",
  "state": { ...full state object as above... }
}
```

Import accepts BOTH this wrapped shape **and** a bare `state` object (for
manual edits). Detection: `parsed.state ? parsed.state : parsed`.

### Rules every future build MUST follow

1. **Schema is append-only.** New fields are added with defaults in
   `ensureShape()`. Never rename, retype, or remove a field that v1-v4
   wrote — older imports must keep loading. If a field's meaning truly
   has to change, give the new meaning a new field name and migrate the
   old field into it inside `ensureShape()`.
2. **Bump `SCHEMA_VERSION` whenever you add a field or a migration.**
   Record the change in the Migration Ladder table above.
3. **`ensureShape()` is the only place migration logic runs.** It must
   be idempotent — running it twice on the same state must be a no-op
   on the second call. It must never throw on a partially-formed save.
4. **Preserve unknown fields with a passthrough.** When migrating, copy
   the entire raw object first, then mutate — so a JSON exported from
   a newer build (with fields the running build doesn't recognize)
   imports without data loss when re-exported. This also applies at
   the VALUE level: `ensureShape` (Build 05252026.4+) deliberately
   passes through any unknown string in `quests[q]` instead of
   coercing it back to a known value. If a future build adds a new
   completion mode (say `"raid"`), older builds will still surface
   that quest as a generic "done" and re-export it intact.
5. **`saveState()` strips ONLY known transient fields** (currently just
   `puzzleEditing`). Everything else is durable.
6. **Export round-trips losslessly.** Anything visible in the app that
   the user can change MUST be in `state` and therefore in the export.
   New user-editable surfaces (themes, puzzles, notes, etc.) are not
   "done" until they're round-tripping through Import.
7. **`STORAGE_KEY` is permanent.** Never bump it; bump `SCHEMA_VERSION`
   instead. Two legacy keys exist for read-only fallback during
   first-load migration: `sooksSagaScroll.v1` (one-time) and the
   current `sooksSagaScroll`. Master Reset clears both.
8. **Import confirmation is non-negotiable.** Import must prompt the
   user before replacing in-memory state (`confirm()` dialog with
   character count). No silent overwrite.

### Quick mental model for adding a feature

Adding a new piece of user data (e.g. a per-saga colored flag)?

1. Decide the field's name and shape; document it in the schema block
   above.
2. Add a default to `ensureShape()` (e.g. `if (!ss.flag) ss.flag = null;`).
3. Bump `SCHEMA_VERSION` and add a row to the migration ladder.
4. Wire the UI to read/write the field; call `saveState()` after writes.
5. Verify Export + Import round-trip: write, export, master-reset,
   import, confirm the field came back.

## Known gaps / to confirm
- **All SAGA_STORIES filled (Build 05242026.1).** 29 of 29 sagas have
  wiki-sourced taglines. Where saga pages had no synopsis (Cogs, Gianthold,
  Cormyr Heroic, Huntsilvers Heroic), pack-page descriptions were used with
  proper citation.
- **Gianthold quest givers filled (Build 05242026.1).** All 19 previously
  "Unknown" entries across h-gianthold and e-gianthold now have wiki-sourced
  bestowers. Wilderness walk-ups use the format `Walk-up (end reward: NPC)`
  to capture both the entrance-bestow mechanic and the turn-in NPC.
- **e-planeswalker partial flag cleared (Build 05242026.1).** The wiki shows
  the saga consists of 11 quests (Beyond the Rift + The Lost Thread + 3
  Queen of Demonweb + 5 Netherese Legacy + Through a Mirror Darkly). The
  previous 9-quest list incorrectly included Shadow Over Wheloon quests
  (Friends in Low Places, A Lesson in Deception, etc.) that aren't actually
  in the wiki's requirement list. Full quest list, NPC info (Eclesta
  Xillian or Istine Madrovelle), pack list, storyArcs, and QUEST_GIVERS
  rebuilt from `https://ddowiki.com/page/The_Planeswalker's_Path_(Epic)`.
- **storyArc audit (Build 05242026.1).** Saltmarsh main arc, Three-Barrel
  Cove, Lord Arden's Slumber, Hyrsam and the Needle, Curses of the Kopru,
  Isle of Adventure, Chill Part One/Two, Wastes of Gianthold, Return to
  Gianthold, and Druid's Deep verified against wiki — all genuinely have
  no single chain bestower NPC (each quest is bestowed individually).
  `giver: null` is the correct representation.
- **Pirates of the Thunder Sea audit + corrections (Build 05242026.26).**
  User flagged in-game progression mismatch: Taggart d'Deneith's chain
  starts with Lille d'Deneith (Bargain of Blood), not Tyn (Black Loch),
  and Cassiopia Carragher was not bestowing any of the per-quest
  e-pirates quests. Wiki re-verification confirmed both:
    - **Sentinels of Stormreach chain order (heroic & epic):**
      Bargain of Blood (Lille) → The Black Loch (Tyn) → Storm the
      Beaches (Galton) → The Tide Turns (Greigur). `storyArcs[0].quests`
      arrays reordered to match in both h-pirates and e-pirates.
    - **Spies in the House is NOT part of the Sentinels chain** (wiki:
      "this quest isn't part of the chain and is strictly optional. It
      is, however, required for the Epic The Pirates of the Thunder Sea
      Saga"). Bestowed by **Brysen d'Deneith**, not Cassiopia. Moved
      into its own `storyArcs` entry labeled "Side Quest (Sentinels
      pack — not in chain)".
    - **h-pirates QUEST_GIVERS fabrications replaced.** Drell Jake had
      been listed for all 4 Three-Barrel Cove quests beyond Tobias;
      wiki bestowers are **Dirty Vingus** (Prove Your Worth), **Marteen
      Worley** (Ghost of a Chance), **Red Tom** (The Troglodytes' Get
      AND Old Grey Garl — both Garl's Tomb quests). **Laurent Chastel**
      bestows Two-Toed Tobias (already correct).
    - **e-pirates QUEST_GIVERS fabrications replaced.** Cassiopia
      Carragher had been listed for all 5 TBC epic quests + Spies; wiki
      bestowers are **Fearless Frida** (A Legend Revisited), **Tarfoot
      Brynne** (Old Tomb, New Tenants), **Dirty Vingus** (Prove Your
      Worth Epic), **Marteen Worley** (Ghost of a Chance Epic), **Niles
      Cage** (Precious Cargo), **Brysen d'Deneith** (Spies in the
      House). Cassiopia is the SAGA giver only.
    - **Taggart's location refined** to "House Deneith Enclave, at the
      base of Deneith Tower" (wiki-accurate vs. earlier "in front of
      Sentinels Tower").
  Same fabrication pattern — one NPC repeated across many quests —
  likely exists in other sagas (h-cogs Cassana McCallister, h-saltmarsh,
  some Legendary sagas) and should be audited the same way: per-quest
  bestowers should match the per-quest wiki infobox, not the saga giver.
- **QUEST_DETAILS coverage essentially complete (Build 05242026.4).** Added
  10 new buckets covering 120 quests across this session: h-cogs (8
  walk-ups), h-pirates Three-Barrel Cove (5), h-feywild (13), h-isle-dread
  (12), h-chill (13), h-mist (12), h-myth-drannor (13), h-gianthold (10),
  h-masterminds (10), h-cormyr (24). h-huntsilvers and e-huntsilvers resolve
  via cross-saga lookup against h-cormyr's 16 shared quests; e-eberron's 3
  quests resolve via e-menace; e-cormyr's Cormyr-specific quests resolve
  via h-cormyr; all Legendary tiers (l-mists, l-cogs, l-masterminds,
  l-feywild, l-saltmarsh, l-isle-dread, l-myth-drannor, l-chill, l-vecna)
  resolve via `lookupQuestDetails()` cross-tier fallback. Net result: every
  unique normalized quest now has wiki-sourced details.

## Resume prompt
> I'm resuming Sook's Saga Scroll at `~/ClaudeGarage/personal/sooks-saga-scroll/`. It's
> a single-file offline HTML saga tracker for DDO — SIX tiers as of
> Build 06082026.2: **Heroic**, **Non-Saga Heroic** (FIVE per-pack
> Harbor containers as of Build .4 — Baudry Cartamon, The Lost Seekers,
> Harbinger of Madness, Web of Chaos, and Harbor Standalones; 37
> wiki-listed Harbor quests, L2–17, for early leveling), **Non-Saga
> Epic** (Phiarlan Carnival's
> 4 quests + Time Flies — for leveling at L20), **Epic**,
> **Legendary** (the 29 original sagas), and **Non-Saga Legendary**
> (21 standalone Legendary containers, 86 quests, for First Time
> Reaper). Group order: Heroic → Non-Saga Heroic → Non-Saga Epic → Epic →
> Legendary → Non-Saga Legendary. Per the data self-check the build now has
> **64 containers / 541 quest slots** (all wiki-detailed) — includes Update 81
> "Terror of Demogorgon" (Heroic `h-demogorgon` L11 + Legendary `l-demogorgon`
> L37, 12 quests each) as of Build 07222026.1. Multi-character
> with
> add/rename/delete and per-character themes (Scholar / Soldier / Ranger /
> Barbarian / Rogue / Divine / Arcane / Necrotic / Knight). Progress saves
> to browser storage under key `sooksSagaScroll` (**schema v14** — see the
> Data Schema & Import/Export Contract section of the README and follow it
> exactly; new fields go through `ensureShape()`, never rename or remove
> a field that older builds wrote, always bump `SCHEMA_VERSION`, never bump
> `STORAGE_KEY`). Import/Export must round-trip every user-editable field
> losslessly — that's the user's only backup path.
> Read the latest `sooks-saga-scroll-<MMDDYYYY>-<N>.html` (full kebab-case +
> build suffix; folder keeps the latest 3 build files) to load the
> current state. Key mechanics:
> `SHARED_QUESTS` is keyed by `tier::questName` so cascade is within-tier
> only; `lookupQuestDetails()` falls back across tiers (and via
> `normalizeQuestName()` strips `(Epic)` / `(Legendary)` suffixes, a
> leading `Return to ` prefix, **and** a leading `The`/`A`/`An` article
> — so the full Heroic ↔ Epic Gianthold pair resolves 10-for-10) so
> detail data is shared everywhere. Wiki-sourced data lives in
> `SAGA_STORIES`, `QUEST_DETAILS`, `QUEST_GIVERS`; never fabricate — use
> Chrome MCP against `ddowiki.com` (direct `web_fetch` is blocked by the
> proxy). Footer uses `Build ` prefix with `mmddyyyy.x` format,
> currently `Build 07222026.1`. **Build 07222026.1 (still v14; data-only): DDO
> Update 81 "Terror of Demogorgon" added as two containers — `h-demogorgon`
> (Heroic L11) between `h-mist`/`h-myth-drannor`, `l-demogorgon` (Legendary L37)
> at the bottom of Legendary — 12 quests each in 3 arcs; single
> `QUEST_DETAILS["h-demogorgon"]` block, legendary resolves via fallback (12/12).
> Verified in-browser: 0 self-check errors, 64 containers / 541 quests / 100%
> detailed. Preview-data gaps (4 synopses, 2 locations, giver spelling, full
> monster lists) flagged `TODO(u81-resource)` for the U6 live re-source (dossier:
> `data/update-81-terror-of-demogorgon-research.md`; plan:
> `docs/plans/2026-07-22-001-feat-update-81-terror-of-demogorgon-plan.md`).**
> **Build 07182026.9 (still v14): CPU/animation
> performance pass — the continuous progress-bar sheen (~121 `background-position`
> animations at once) now runs only on COMPLETE bars (`.all-done` / reaper
> `.sealed`) via a GPU `transform`; partial bars get a static gold shine. The
> data-scaling box-shadow pulse glows now pulse a GPU `opacity` overlay (bar
> pulses anchored off the `overflow:hidden` bar). Measured before/after in Chrome:
> running sheens 121 → 8, all pulses animate `opacity` not `box-shadow`. Verified
> in-browser by Claude.**
> **Build 07182026.8 (still v14): pill + LFM
> polish — the reward pill's Banked/Incomplete states are now the same width
> (sized to "INCOMPLETE"), and the LFM notice stands out more (gold-accented
> neutral header bar + bolder quest live-line). Verified in-browser by Claude.**
> **Build 07182026.7 (still v14): two
> visual-pass fixes — the reward pill now bottom-aligns with the progress bars
> at the same height, and the reaper bar's count numbers are cream-on-dark-halo
> for contrast over the dark-red fill.**
> **Build 07182026.6 (still v14): header/level
> polish — the container header's LFM notice became a full-width centered bar
> (was a right-justified chip); the level tag moved to the LEFT of the name on
> saga headers and quest rows; and level tags are now uniform width.**
> **Build 07182026.5 (still v14): code-review
> fixes for the .4 roll-over — the critical one being that .4's container
> queries never fired (`container-type` was on the same element the
> `@container` rules targeted; a container queries its descendants, not
> itself), so nothing rolled; moved `container-type` to the `.saga` /
> `.quest-block` wrappers. Plus a11y tab-order restore and a quest threshold
> bump (560→640).**
> **Build 07182026.4 (still v14): roll-over
> UNIFORMITY rework to CSS grid-template-areas (superseded by .5's container
> fix). Build 07182026.3 (still v14): uniform
> roll-over row stacking — saga container headers and quest rows share one
> 3-tier system (name+level / actions / LFM); superseded by .4's grid rework.**
> **Build 07182026.2 (still v14): CSS-only
> code-review follow-up — the narrow-stack Server divider + "Server" label
> are gated on `:has(.corner-panel:not(:empty))` so they no longer dangle
> over empty space when no server is set.** **Build 07182026.1 (still v14): narrow
> windows (≤1179px) reflow the two corner clusters into one labeled
> priority column (Filters → Characters → LFM → Server) via a matchMedia
> reparent into `#narrowStack`, restoring to `<body>` when wide so the
> desktop layout is unchanged; each saga/non-saga header swaps the FTR
> badge + lone bar for two equal captioned bars (First-Time Reaper +
> completion) on the far right; and the global `#progressBand` summary
> band was removed (with its dead `renderProgress`/`charSummary`).**
> **Build 07142026.2 (still v14): the
> Filters & Alerts fold bar is styled as a real button (control chrome +
> wax-seal caret coin + always-visible "click to fold/open" hint) and
> the tier button grid left-aligns under its TIER label
> (justify-content: flex-start) to match the Group by / Search
> label-over-control pattern.** **Build 07142026.1 (schema v13 → v14,
> Quest Autocomplete + panel roll-up): the 15s live poll auto-marks the
> quest the active character is standing in (location_id →
> getQuestAreaMap, the guild roster's area→quest table) when the
> per-character setting is ON — additive `characterAutoComplete` map
> ({ on, mode }); "normal" = green Quest Completed seal only (sagaDone
> cascade + auto-bank via standalone `_acApply`), "reaper" = also seals
> First-Time Reaper (☠ button cascade, NO_REAPER falls back); fires on
> entry once per stay (`_acSeen`), tier picked by `_sagaForQuest` +
> persisted level, `.ac-toast` announces, defaults OFF + Non-Reaper;
> controls live in a new Quest Autocomplete row under Audio Alerts.
> The whole filter/alert panel now ROLLS UP via a "Filters & Alerts"
> header bar (#ctlFold; controlsFolded in the META key).**
> **Build 07122026.1 (schema v12 → v13,
> four fixes): live refresh PAUSES while the Add Character / Manage
> panel is open (liveRefreshTick returns early on charManageMode — its
> renderChars() was wiping half-typed input every 15s; catch-up tick on
> close); Audio Alerts are PER-CHARACTER via the additive
> `characterAudio` map (checkbox writes seed from resolved prefs then
> diverge; falls back per-field to legacy global META audioCues, then
> defaults High+Raids ON; follows rename/delete; Export round-trips;
> syncAudioChecks() runs from renderAll); RESPONSIVE pass — ≤1179px the
> fixed corner clusters drop into the document flow as wrapping card
> rows, 1180–1620px the scroll cedes side gutters, fixed control widths
> became caps, ≤480px phone tightening (zoom shrinks the CSS viewport,
> so the same queries cover zoom collisions); and the DDO Forums thread
> is linked beside the creator credit (.credit-forum).**
> **Build 07112026.18 widened the
> Suggested migration: legacy `band:*` tierFilter values migrate to
> "suggested" unconditionally (band values can only be legacy chip
> output; rescues saves that already booted .17); "all" migrates once;
> explicit tiers respected.** **Build 07112026.17 replaced the
> level-suggestion chip with a gold "Suggested" tier-filter button
> (default; additive tierFilter value "suggested"; dynamic band
> resolution via `resolveTierFilter`; `VALID_TIER_FILTERS` gained the
> value — the validator otherwise stomps it to "all"; one-time
> `suggestedDefaultApplied` META migration).** **Build 07112026.16 is a design-cohesion
> pass: `:root` tokens — radii (`--radius-card/ctl/tag/pill`), one
> `--frame-line` hairline, 3-step shadows (`--shadow-card/ctl/lift`),
> and font roles (`--font-display/body/flavor`) — replace 14 radii /
> 30 sizes / 58 shadows / 9 font stacks; sizes snap to 0.64–0.95 scale;
> future CSS should use these tokens, never raw values.**
> **Build 07112026.15 added the Bonus Days
> banner, GIST edition: `refreshBonusBanner()` fetches the user-owned
> gist (`DDO_BONUS_GIST_URL`, eddiefiggie/0cf713…, raw unversioned form,
> CORS-open — verified live) on every load; shows headline/details while
> `expires` isn't past, hides on failure/expiry; update = edit the gist
> or ask Claude, the HTML never changes.** **Build 07112026.14 REMOVED the bonus
> banner after live QA proved no automatable source exists for a local
> file (ddowiki api.php/rest.php firewalled HTTP 202 even same-origin;
> page HTML has the table but no CORS header; DDO Audit's 66-route API
> has no bonus data) — and upgraded fetchServerPopulation to the live
> `/v1/population/timeseries/day` newest-datapoint character_count
> (primary) + who-list fallback, with a `!= null` guard so a missing
> server can't coerce into a fake "0 online".** **Build 07112026.13: the daily-check
> stamp only short-circuits when an ACTIVE coupon is on record — empty
> results retry every load (an empty stamp used to block all later
> checks until midnight, hiding the .12 fixes).** **Build 07112026.12 hardened the banner:
> expiry-only coupon rows (single date) now parse as already-started
> (latest expiry wins among active rows), a JSONP fallback (`_wikiJsonp`)
> rescues the wiki check when CORS fetch fails, and a hidden banner
> console.warns its reason; `ddowiki-connectivity-test.html` (unversioned)
> diagnoses the two transports standalone.** **Build 07112026.11 added expiry-kill to
> the bonus banner: `_extractActiveCoupon` returns `{text, expiry}`
> (META `couponExpiry`), `_couponActive()` hides the banner once the
> event lapses and blocks expired carry-forward when the wiki is down;
> the daily check then watches for the next event. ddo.com/news URL
> probing ruled out (JS-shell pages + no CORS).** **Build 07112026.10 made the bonus
> banner AUTOMATED-ONLY: the manual Discord-pasted line + ✎ editor are
> gone; the banner shows just the wiki-sourced coupon and hides when no
> automated data exists. (Check ddowiki.com/page/Bonus_Days manually —
> if it tracks the weekly bonus, that half can be automated the same
> way.)** **Build 07112026.9 added the DAILY BONUS
> banner under the subtitle (`#bonusBanner`, META `dailyBonus`): the
> in-game bonus line is user-set via ✎ (Discord can't be read by a local
> file), the Bonus Days coupon auto-pulls once per calendar day from the
> DDO Wiki Coupons page via MediaWiki api.php `origin=*`
> (`_extractActiveCoupon`, `refreshDailyBonus`, `checkedDay` guard,
> ↻ force, self-diagnosing tooltip; wiki CORS unverified in the
> no-egress sandbox — degrades silently).** **Build 07112026.8 made the footer data
> credit legible: `.credit-data` opts out of the footer's italic/faded
> base (upright EB Garamond 0.92rem, `--ink-secondary`, bold wax-red
> solid-underline links).** **Build 07112026.7: FIRST TIME REAPER
> spotlight in the reaper cards (`_ftrNeeded` → `tr.ftr-need` gold pulse
> + "✦ First Time Reaper ✦" banner, FTR rows sort to the top; clears
> when the quest's per-quest state hits `"reaper"`); DDO Audit credit
> line in the footer (`.credit-data`, links ddoaudit.com + Clemeit);
> server population bar above the guild card (`#serverPop`,
> `fetchServerPopulation` — `/game/server-info` first, who-list count
> fallback, 30s `_popCache`); and `LIVE_REFRESH_MS` doubled to 15000.**
> **Build 07112026.6: quest/raid names in
> every LFM card are DDO Wiki links (`_wikiHref`, spaces→underscores,
> new tab); the reaper stack is THREE cards — Low R1–3 / Mid R4–7 /
> High R8–10 (`renderReaperCorner` over one `_reaperLfms` list, R1+
> collected, `_reaperSig` guard) above Raids; ALL LFM cards default
> COLLAPSED (`_foldDefault`: unset META = folded); and an Audio Alerts
> checkbox row under the tier filter (Low/Mid/High/Raids) gates each
> cue at play time via `_audioPrefs()` (META `audioCues`; defaults
> High + Raids ON, Low + Mid OFF; per-band seen-sets stay maintained
> while off).** **Build 07112026.5 added the saga /
> story-pack line to the High Reaper card: under each quest name,
> `.rd-saga` shows the scroll container the quest belongs to
> (`_sagaForQuest` scans SAGAS by normalized name, tier-band
> disambiguation via the group's posted range; saga `name` for true
> sagas, `pack` for non-saga containers) or falls back to the quest's
> `required_adventure_pack` from DDO Audit (quests ref cache rows now
> carry `pack`, shape-guarded; `_questPackMap`).** **Build 07112026.4 moved the Guild card to
> its own upper-LEFT cluster (`.data-corner.left`) and added the HIGH
> REAPER card (`#highCorner`) above the Raids card on the right
> (Characters → High Reaper → Raids): level-relevant ☠ R8–R10 QUEST
> groups (raids excluded), same 3-line rows but fraction over the
> 6-player quest cap, always .high-tinted, `highFolded` in the META key;
> `refreshLfmIndex` collects `highs` alongside `raids` (`_highLfms` /
> `_highSig`) and the reaper knell's key set derives from that same
> list.** **Build 07112026.3 made three UI cleanups:
> raid rows are three stacked lines (`.rd-name` / `.rd-meta` range ·
> difficulty · ☠ R<n> / `.rd-live` leader · fill/12); the per-tier
> progress-bar band under the header counters is REMOVED (`#tierRollup`
> markup + `tierRollup()` + all `.tier-rollup*` CSS — counters and
> per-saga header bars stay); and the filter/search controls are one
> bordered panel with a single button-width standard (`--ctl-w: 172px`,
> keyed to "Non-Saga Legendary"), the tier filter a centered flex-wrap of
> fixed-width buttons that collapses on narrow screens (never widens),
> modern 9px-radius hover-lift styling, all control hooks unchanged.**
> **Build 07112026.2 left-justified the
> guild/raids title bars (`.guild-table thead th` text-align left) and
> added audio cues: a low tritone knell when a new level-relevant HIGH
> reaper (☠ R8–R10) quest group appears, a rising horn when a new
> level-relevant raid group appears (R8+ raids ring the horn only).
> Novelty = (quest, leader) vs the previous pass in `refreshLfmIndex`
> (`_seenHighLfms`/`_seenRaidLfms`; fill churn never re-triggers; first
> pass seeds silently); relevance shares `_lvlRelevant()`/
> `_activeCharLevel()` with the raids card; WebAudio-synthesized, lazy
> AudioContext with one-time gesture resume, all try/caught.**
> **Build 07112026.1 added the RAIDS card:
> a third collapsible corner card (`#raidCorner`) under the Guild panel
> listing live raid LFMs relevant to the active character's level. The
> quests ref cache rows now carry `raid` (group_size === "Raid"; shape
> guard refetches older caches once), refreshLfmIndex splits raid LFMs
> into `_raidLfms` on the same fetch/30s tick (own signature guard
> `_raidSig`; raids also stay in the quest badge index), and
> renderRaidCorner filters by the persisted aggregate level — posted
> range must include it, no range → shown, level unknown → all shown.
> Row: raid name · range, then "☠ R8 · <leader> · <party>/12" (12 = raid
> cap; blank leader → Anonymous; .mid/.high tints). Folds like the guild
> card; `raidFolded` lives in the META key.** **Build 07052026.12 made LFM matching
> tier-aware: the index stores per-group records (`entry.groups` with
> minL/maxL/diff/skulls/leader/party) and `lfmForTier(questName, tier)`
> aggregates only groups whose level range overlaps the container's band
> (LFM_TIER_BANDS heroic 1–19 / epic 20–29 / legendary 30+) — heroic groups
> no longer light Epic/Legendary copies of a quest and vice versa. Groups
> with no posted levels show in all tiers; ranges straddling a band edge
> match both.** **Build 07052026.11 fixed the ☠ tag looking
> partially covered: the flush attachment sat inside the bar's glow halo and
> under its rounded corners; now a 4px cell gap, full 8px rounding, restored
> top border, and position:relative + z-index:1 keep the tag above any glow
> — a clean label plate under the bar.** **Build 07052026.10 integrated the LFM info
> elegantly: quest rows are a structured two-line layout — `.quest-main`
> column with `.quest-line1` (name · shared · giver · level) and a dedicated
> `.quest-live` sentence-style line beneath ("🔥 R5 · Henk's group · 3/6 ·
> lvl 5–8", tier-tinted, fit-content, slim left rule) rendered only when a
> live group exists; the inline `.quest-lfm` chip is gone. The header ☠ tag
> is an attached label under the progress bar (130px, bottom-only radius, no
> top border, gap 0).** **Build 07052026.9 fixed the overlapping
> quest-row text caused by .8's wider badge: `.quest` now flex-wraps (chips
> drop to a second line; the name no longer collapses under rigid nowrap
> chips), `.quest-lfm` wraps internally (max-width 100%), and the header ☠
> tag — previously an illegal 7th item in the 6-column .saga-header grid —
> now stacks under the progress bar inside `.saga-progress-cell`, wrapping
> within the 130px column.** **Build 07052026.8 added the group leader's
> name + party fill to the LFM indicators: index entries carry `leader` (LFM
> poster; blank/anonymous → "Anonymous") and `party` (leader + members, vs
> the 6-player cap), taken from the group with the highest declared skulls
> and folded into the repaint signature. Header tag: "☠ R8 · Henk · 3/6"
> (leader half in plain EB Garamond `.lfm-lead`); quest badge: "🔥 1 LFM ·
> R5 · Henk 3/6". Also live-confirmed the flagging works identically for
> non-saga containers.** **Build 07052026.7 amplified the reaper-tier
> signal: strong orange/red wash inside the container bar + hot borders
> (#ff8c1a/#ff2f1f) + wide halos with BOTH tiers pulsing (lfmMidPulse/
> lfmHighPulse, reduced-motion-gated); a small header tag `.lfm-skull-tag`
> ("☠ R<n>") beside the bar for ANY declared count (.low R1–3 neutral, .mid
> orange, .high red pulsing); and matching quest-row glow inside opened
> containers (`.quest.lfm-mid-row`/`.lfm-high-row`, inset-left-edge shadow so
> layout doesn't shift, red pulses via lfmRowPulse) riding renderQuestRow's
> lfmHit.** **Build 07052026.6 added reaper-tier LFM
> highlights — `refreshLfmIndex` parses comment-declared skulls (R1–R10,
> `\br\s?(10|[1-9])\b`, kept as `skulls` per index entry + in the repaint
> signature; DDO never broadcasts the count, comments are the only source) and
> the container progress bar tints by the highest declared tier among its
> quests: R4–R7 brilliant orange (.lfm-mid), R8–R10 brilliant red (.lfm-high,
> pulsing, reduced-motion-gated), R1–R3/undeclared no tint; 🔥 badges show
> "· R<n>". It also added a 30-second live refresh: LIVE_REFRESH_MS polling
> gated by the Page Visibility API (hidden = skip, visibilitychange = instant
> catch-up), each tick force-expiring _ddoCache/_guildCache/LFM window and
> re-rendering medallions + guild corner + LFM index (signature-guarded, no
> idle DOM churn). DDO Audit has no push channel, so polling is the model.**
> **Build 07052026.5 renders anonymous
> guildmates as a faded-italic "Anonymous" label (`.gm-anon`, tooltip) instead
> of "?" — DDO Audit blanks the `name` while a player's in-game Anonymous
> flag is on (single-char endpoint 403s); `_summarizeGuild` carries an `anon`
> flag (is_anonymous OR blank name), level/class kept. It also made progress
> bars BRILLIANT when a container has any progress: `has-progress`
> (done > 0) = gold border + glow + green→gold fill + sheen animation
> (behind prefers-reduced-motion); `all-done` (done >= total) glows warmest.
> Empty bars stay quiet parchment.** **Build 07052026.4 made the guild card
> foldable: clicking the guild-name header (or Enter/Space; `role=button`,
> `tabindex=0`) collapses the roster to just the header bar (caret ▸/▾, no
> tbody rendered, ref resolution skipped while folded). Fold state persists in
> the META key `sooksSagaScrollMeta` (outside `state` — no schema change) via
> merge-style `_readAppMeta()`/`_writeAppMeta()`; the Export handler's
> `lastExportedAt` write now merges through `_writeAppMeta` instead of
> overwriting the meta object. New CSS `th.gm-fold` + `.gm-caret`.**
> **Build 07052026.3 is a guild-box legibility
> pass (CSS only): roster body 0.74 → 0.85rem, meta + quest/zone lines 0.7 →
> 0.8rem, faded ink replaced with full-strength `--ink` on member lines
> (gm-empty stays faded), italics dropped on the quest/zone line, header
> 0.76rem, line-heights/padding opened up, and `.gm-quest` min-height retuned
> to `calc(0.8rem * 1.3)`. No markup/JS change; overflow-safety and
> uniform-thickness rules retained. No schema change (still v12).**
> **Build 07052026.2 made the roster's second
> line ALWAYS render with a reserved line of height (min-height on
> `.gm-quest`) so every member row is the same thickness, and made it show
> the member's ZONE (public spaces included, "📍 <zone>", class `.gm-zone`,
> via `getAreasMap()` — same 7-day ref cache) whenever they're not inside a
> quest; in-quest stays "⚔ <quest>". Ref tables resolve in one `Promise.all`
> with a stale-guard. No schema change (still v12).** **Build 07052026.1 made each guild-roster row
> two stacked lines: line 1 = name · heroic class(es) · level (meta reordered
> class-first, e.g. `Fighter/Monk · Lvl 34`), line 2 = the quest that member is
> running (⚔-prefixed, italic), shown only when their `location_id` resolves to
> a quest instance. The quests ref cache (`sooksSagaScrollRef`) now also stores
> each quest's `area_id` (slim key `area`; old caches without it are treated as
> stale and refetched once), building `_questAreaMap` / `getQuestAreaMap()`;
> `_summarizeGuild` keeps each member's `location_id`. The roster table is
> single-column (one `.gm-cell` per member; header th no colspan). Everything
> wraps inside the fixed 200px card — no nowrap, `overflow-wrap: anywhere` on
> name/meta/quest, `.gm-line1` flex-wraps — so nothing runs off the right edge.
> No schema change (still v12).** **Build 06302026.1 fixed the online-guildmates
> roster table clipping member names at the right edge: `.guild-table` now uses
> `table-layout: fixed` and both `.gm-name` (width 52%) and `.gm-meta` switched
> from `white-space: nowrap` to `white-space: normal` + `overflow-wrap: anywhere`
> so long names and multiclass metas wrap inside the 200px card instead of being
> cut off by the panel's `overflow-x: hidden`. CSS only; `renderGuildCorner()`
> untouched; no schema change (still v12).** **Build 06222026.4 made the contextual
> level-relative "Show" reveal BOTH the saga tier and its non-saga sibling for
> the character's level band (Heroic+Non-Saga Heroic / Epic+Non-Saga Epic /
> Legendary+Non-Saga Legendary), via a `band:<tier>` value of `state.tierFilter`
> (new `TIER_BANDS` + `bandTiers()` / `tierFilterActiveSet()`; the saga filter
> shows all band members, both buttons read active, `VALID_TIER_FILTERS` accepts
> the band values). The chip reads "suggested: <Tier> + Non-Saga <Tier>" / button
> "Show <Tier> (saga + non-saga)" and still folds up sagas. No schema change
> (still v12).** **Build 06222026.3 split the upper-right cluster
> into two clearly-distinct cards — a CHARACTERS card (`.corner-panel.char-panel`:
> "Characters" caption + chip list `#charCorner` + Export/Import) and a separate
> GUILD card (`.corner-panel.guild-panel`, formerly `#guildCorner`, headed by the
> guild name) with a gap between them; both fixed 200px wide to align. It also
> made the level-aware "Show <Tier>" button FOLD UP all sagas (Expand-all off +
> expand/collapse sets cleared) so showing the relative-level tier yields a tidy
> collapsed overview. No schema change (still v12).** **Build 06222026.2 moved
> the guildmates-online display OFF the medallion into a floating TABLE in the
> upper-right cluster (`#guildCorner`, under `#charCorner`): header = the active
> character's guild
> name ("⟨Guild⟩ · N online"), rows = each online guildmate's name + Lvl/class
> (self + offline excluded); sticky header, internal scroll, hidden until the
> active char has a server + guild. New `renderGuildCorner()` (called from
> `renderAll` and `updateCharStats`); reuses the Build .1 live path; no schema
> change (still v12).** **Build 06222026.1 added five DDO Audit
> live-data features (no schema change, still v12), all best-effort /
> offline-first / CORS-safe and keyed off the active character's server,
> with NO new persisted state**: (1) live **location** — resolve a char's
> numeric `location_id` via `GET /v1/areas` → "· 📍 <area>" on the medallion,
> plus a conservative name/pack/region match that highlights the matching saga
> card (`saga-now-playing`) and shows a jump-to banner; (2) **level-aware tier
> suggestion** chip (reuses persisted `characterLevels`: 1–19 Heroic / 20–29
> Epic / 30+ Legendary, with a "Show <Tier>" button); (3) **last seen** on
> offline medallions (`last_update`→`relativeTime()`); (4) **guildmates
> online** via `GET /v1/characters/by-server-and-guild-name/{server}/{guild}`;
> (5) **live LFM badges** — `GET /v1/lfms/{server}` + `GET /v1/quests`
> (quest_id→name, skip 0) build an index matched by `normalizeQuestName()`,
> rendering "🔥 N LFM" on quest rows (60s-window + change-signature guard so
> async re-renders can't loop). DDO Audit reference tables (areas, quests) are
> cached OUTSIDE `state` in a separate localStorage key `sooksSagaScrollRef`
> (7-day TTL) so the schema is untouched; live LFM/guild data caches ~60s in
> memory. Only the NEW servers (Cormyr / Moonsea / Shadowdale / Thrane) carry
> live data — older servers return empty and the features stay inert. The
> DDO Audit OpenAPI lives at `api.ddoaudit.com/docs/openapi.json`; useful
> endpoints beyond the above: `/v1/game/server-info` (server status +
> character_count/lfm_count), `/v1/guilds/{server}/{guild}`,
> `/v1/population/*`, `/v1/quests/{id}`. **Schema is at v12** (v12 adds the
> per-character `characterLevels` map — last-known aggregate level captured
> from DDO Audit and persisted for offline level-based features; medallions are
> fixed-size with a clamped stats line and show "Missing server" when no server
> is set. v11 adds the
> per-character `characterServers` map powering the optional DDO Audit
> live-status line under each medallion — additive, mirrors `characterThemes`,
> round-trips via the whole-state Export; lookup calls the direct DDO Audit
> endpoint `GET https://api.ddoaudit.com/v1/characters/<server>/<name>` (the
> `characters` blueprint is `version=1`, hence `/v1`), reads `{data, source}` +
> `is_online` / `is_anonymous`, renders the snake_case model, and falls back to
> "Offline mode" / "Anonymous" on offline / 404 / 403 / unreachable — isolated in
> `DDO_AUDIT.charUrls` / `describeCharacter` / `fetchCharacter`. v10 adds the
> sixth tier value `"Non-Saga Heroic"`, used by the five per-pack Harbor
> containers;
> storage shape is unchanged from v9 — additive only). Build .4 (v9)
> surgically rewires the renderer so
> `saga.isNotSaga === true` containers (a) don't render the per-quest
> Skip button and (b) label container completion as "Completed" /
> "Incomplete" instead of "Banked" / "Incomplete". The underlying
> `reward` field stays "banked"/"incomplete" — only the user-facing
> label flips. Per-quest "Complete for SAGA" button becomes "Mark as
> COMPLETE" / "Marked COMPLETE" on non-saga rows. Confirm dialogs
> say "container" instead of "saga" and drop the Skip-slots bullet
> on non-saga reset. Schema v9 itself introduces two
> new tier values, `"Non-Saga Epic"` and `"Non-Saga Legendary"`, and a
> new SAGAS-entry flag `isNotSaga: true`. **23 non-saga containers**
> (2 Epic L20 + 21 Legendary) sit alongside the original 29 sagas;
> the renderer suppresses the "Saga Giver" / "Location" rows for
> non-saga containers in favor of an italic "Not a saga" note. Group
> order is **Heroic → Non-Saga Epic → Epic → Legendary → Non-Saga
> Legendary**. Non-saga tiers explicitly do NOT track raids — the
> wiki Raids category page (https://ddowiki.com/page/Raids) is the
> authoritative filter. Non-Saga Epic L20: Phiarlan Carnival (4
> quests, The Maleficent Cabal arc bestowed by Rouge in the House
> Phiarlan Enclave) + Time Flies (Anniversary Celebration, bestower
> Sariyon Dran-dal). Non-Saga Legendary: 86 quests across 21 packs,
> covering everything the wiki Legendary table classifies as non-saga
> + non-raid (Slave Lords, Attack on Stormreach, Disciples of Rage,
> Dragonblood Prophecy, Fall of Night Brigade, Free-to-Play singles,
> Grip of the Hidden Hand, Hunter and Hunted, Illithid Invasion,
> Planar Eyes, Slice of Life, Tavern Tales, Devil's Gambit, Dragons'
> Hand, Lost Gatekeepers, Tethyamar, Soul Splitter, ToEE (two arcs —
> Elmo + Canon Terjon, with 7 per-quest bestowers), Vale of Twilight,
> Trials of the Archons, White Plume Mountain). Migration v8→v9 is
> purely additive (no data fields changed); older builds reset
> state.tierFilter to "all" if they see an unrecognized value,
> otherwise user progress is intact. v8 (the
> previous version) adds
> `skipDone[questName] = true` per saga, splitting Skip off from
> `quests[q]` so Reaper and Skip are now independent (a quest can be
> marked both). Migration auto-mirrors any legacy `quests[q] === "astral"`
> into `skipDone[q] = true`. v7 (before that) adds
> `state.questOverrides[questName] = { path?, traps?, survival? }` so
> users can replace the rendered Path / Traps / Survival rows; falls
> back to wiki when an override is absent. Builds on v5's `state.tierFilter`
> (page-level tier filter) and a 4th quest completion value `"astral"`
> (the user-facing Shard / VIP Skip button — internal value name stays
> `"astral"` for durability; UI label is "Skip" / "Skipped").
> Theme labels are colors now (Carmine, Oxblood, Moss, Crimson, Slate,
> Sapphire, Amethyst, Bilegreen, Silver) but `THEMES[i].id` stays
> archetype-named — durable schema key, presentational label.
> Then help me with: [describe the change].

---
_Last updated: 2026-07-18_ (live build **07182026.9** — CPU/animation performance pass: the app ran Chrome hot because ~121 progress bars each animated a continuous `background-position` "sheen" (one raster repaint per frame). The traveling shimmer now runs only on COMPLETE bars (completion bar `.all-done`, reaper bar `.sealed`) via a GPU-composited `transform`; partial bars get a static gold shine; and the data-scaling box-shadow pulse glows now pulse a GPU `opacity` overlay (bar pulses anchored off the `overflow:hidden` bar, per a code-review finding). CSS only, still v14. Verified with `node --check` AND a before/after in-browser measurement by Claude — running sheen animations dropped **121 → 8** (only complete bars, now animating `transform` not `background-position`); all four converted pulses animate `opacity` with zero running `box-shadow` animations; visual split + `.5`–`.8` regression + clean console confirmed. Ran the full Compound Engineering loop (brainstorm → plan → doc-review → work). Prior build **07182026.8** — pill + LFM polish: the reward pill's Banked and Incomplete states are now the same width (min-width sized to fit "INCOMPLETE" so "BANKED" pads to match); and the LFM notice stands out more — the container header's neutral LFM bar gets a gold accent border + stronger fill, and the quest row's live-LFM line gets bolder text + a stronger fill + a thicker left rule. Verified with `node --check` AND an in-browser visual pass by Claude. Prior build **07182026.7** — reward-pill/bar alignment + reaper-text contrast fixes. Prior build **07182026.6** — header/level polish: LFM notice became a full-width centered bar, level tag moved left of the name (saga + quest), uniform level-tag widths. Prior build **07182026.5** — code-review fixes for the .4 roll-over: the critical one being that .4's container queries never fired because `container-type` was on the same element (`.saga-header` / `.quest`) that the `@container` rules targeted — a container queries its descendants, not itself — so nothing rolled; moved `container-type` to the `.saga` and `.quest-block` wrappers. Also restored linear keyboard tab order and raised the quest roll threshold 560→640px. Caught by a two-reviewer `ce-code-review`. Prior build **07182026.4** — grid-template-areas uniformity rework (superseded; container queries were mis-targeted). Prior build **07182026.3** — uniform roll-over row stacking (first cut; flex-wrap approach, superseded by .4). Prior build **07182026.2** — CSS-only code-review follow-up gating the narrow-stack "Server" label with `:has(.corner-panel:not(:empty))` so it doesn't dangle over empty space when no server is set. Prior build **07182026.1** — narrow-window priority stack (matchMedia reparent of the corner clusters into `#narrowStack`) + header twin bars (First-Time Reaper + completion) + removal of the global `#progressBand` summary band; built via the Compound Engineering pipeline, plan at `docs/plans/2026-07-18-001-feat-responsive-stack-and-header-bars-plan.md`; no schema change, still v14. Prior live build **07142026.2** — fold-bar button styling + TIER label alignment; the 07142026.1 park brought Quest Autocomplete + the filter-panel roll-up, schema v13 → v14; full ladder entries under **Files** above. Historical entry from the 2026-07-05 park follows: live build **07052026.12** — LFM matching is tier-aware; JS only, no schema change (still v12). The name-normalized index used to light every tier's copy of a quest; now `entry.groups` keeps each group's record ({minL, maxL, diff, skulls, leader, party}, sig from group records) and `lfmForTier(questName, tier)` aggregates only groups whose posted level range overlaps the container's tier band (`LFM_TIER_BANDS`: heroic 1–19, epic 20–29, legendary 30+; Non-Saga tiers share the bands). Both consumers (renderSagaCard container scan, quest-row lookup) switched, so bars/☠ tags/live lines/row glows appear only in the tier actually being run. No-level groups show in all tiers; band-straddling ranges (17–20) match both. Verified headless (jsdom, 74 checks, ALL PASS): lvl 1–4 R4 → exactly one lit container (Heroic Saltmarsh) + one glowing row; lvl 30–34 R8 → exactly the Legendary Sinister Secret of Saltmarsh ("☠ R8 · Xokotli · 2/6"); 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.12`, all Build .1–.11 behaviors intact. Park retention: kept `sooks-saga-scroll-07052026-12.html` + `sooks-saga-scroll-07052026-11.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-10.html` (removed 2026-07-05). Prior build 07052026.11 (superseded) — ☠ tag no longer looks covered by the progress bar; CSS only, no schema change (still v12). Build .10's flush attachment (gap 0, square top, no top border) sat inside the bar's box-shadow glow halo (has-progress / lfm-mid / lfm-high spreads 3–9px) and under its rounded bottom corners, so the halo painted over the tag's top edge. Fix: 4px cell gap, full 8px rounding, top border restored, position:relative + z-index:1 — the tag paints above any glow as a clean label plate. Verified headless (jsdom, 71 checks, ALL PASS): 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.11`, all Build .1–.10 behaviors intact. Park retention: kept `sooks-saga-scroll-07052026-11.html` + `sooks-saga-scroll-07052026-10.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-9.html` (removed 2026-07-05). Prior build 07052026.10 (superseded) — LFM info integrated properly; markup + CSS, no schema change (still v12). Replaces Build .9's free-wrapping chips (functional but patchwork, per user feedback) with a structured two-line quest row: controls left, `.quest-main` column holding `.quest-line1` (name · ↔ shared · giver · level chip) and a dedicated `.quest-live` line only when a live group exists — sentence-style "🔥 R5 · Henk's group · 3/6 · lvl 5–8" in EB Garamond, fit-content width, slim left rule, tinted neutral/orange(.mid)/red(.high); multi-group quests append "N groups"; the inline `.quest-lfm` chip is removed from the name line. Header ☠ tag became an attached label under the progress bar: exactly 130px (the bar's width), bottom-only corner radius, border-top removed, cell gap 0 — bar + tag read as one element. Verified headless (jsdom, 71 checks, ALL PASS): live line renders R4/Henk's group/3/6 with `.mid`, sits inside `.quest-main` under `.quest-line1`, zero inline chips remain, tag geometry assertions hold, 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.10`, all Build .1–.9 behaviors intact. Park retention: kept `sooks-saga-scroll-07052026-10.html` + `sooks-saga-scroll-07052026-9.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-8.html` (removed 2026-07-05). Prior build 07052026.9 (superseded) — layout fix: quest-row text no longer overlaps; CSS + one markup wrapper, no schema change (still v12). Two root causes from Build .8's wider LFM badge: (1) `.quest` was a non-wrapping flex row with rigid right-side chips (nowrap + flex-shrink:0) — chips outgrew the row, the quest name collapsed to 0 (min-width:0) and chips painted over the text; `.quest` now flex-wraps (gap 4px 10px) and `.quest-lfm` wraps internally (white-space normal, overflow-wrap anywhere, max-width 100%). (2) The header ☠ tag was a 7th child of the 6-column `.saga-header` grid, overflowing an implicit 20px cell; the bar + tag now share the last column inside stacked `.saga-progress-cell`, tag + `.lfm-lead` wrapping within 130px, all nowrap removed. Verified headless (jsdom, 69 checks, ALL PASS): every ☠ tag structurally inside a `.saga-progress-cell`, `.quest` flex-wraps, badge wraps internally, no nowrap in tag CSS, 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.9`, all Build .1–.8 behaviors intact. Park retention: kept `sooks-saga-scroll-07052026-9.html` + `sooks-saga-scroll-07052026-8.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-7.html` (removed 2026-07-05). Prior build 07052026.8 (superseded) — LFM indicators name the leader + party fill; no schema change (still v12). `refreshLfmIndex` keeps `leader` (LFM poster's name; blank = in-game Anonymous → "Anonymous") and `party` (leader + members[], shown against the 6-player cap) per entry — taken from the group with the highest declared skulls (first wins ties) so the name matches the tier on the tag; both fields are in the repaint signature so the 30s tick updates them live. Header tag reads "☠ R8 · Henk · 3/6" (leader half in plain EB Garamond `.lfm-lead`, inheriting chip color); quest badge reads "🔥 1 LFM · R5 · Henk 3/6"; tooltips carry "<leader>'s group, N of 6". Also live-confirmed this session: the Build .6/.7 flagging works identically for non-saga containers (Harbor Standalones tinted orange + Arachnophobia row glowing in a fixture test — saga and non-saga share the same renderers). Verified headless (jsdom, 64 checks, ALL PASS): "Henk · 3/6", "Anonymous · 1/6", "Sunalriz · 6/6", badge "R4 · Henk 3/6", 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.8`, all Build .1–.7 behaviors intact. Park retention: kept `sooks-saga-scroll-07052026-8.html` + `sooks-saga-scroll-07052026-7.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-6.html` (removed 2026-07-05). Prior build 07052026.7 (superseded) — reaper-tier signal amplified; no schema change (still v12). Three changes riding the Build .6 skulls index: (1) the container-bar tint is much more pronounced — strong orange/red wash inside the bar (background on `.lfm-mid`/`.lfm-high`), hot borders `#ff8c1a`/`#ff2f1f`, wide halo shadows, and BOTH tiers pulse (`lfmMidPulse` 2.2s / `lfmHighPulse` 1.4s, behind prefers-reduced-motion); (2) a small header tag `.lfm-skull-tag` ("☠ R<n>") beside the progress bar names the declared tier for ANY declared count — `.low` neutral parchment R1–R3, `.mid` hot orange R4–R7, `.high` hot red pulsing (`lfmTagPulse`) R8–R10; (3) inside an opened container the exact quest row with the declared-R LFM glows in the same fashion — `.quest.lfm-mid-row`/`.quest.lfm-high-row` (color wash + inset left-edge shadow so layout doesn't shift; red pulses via `lfmRowPulse`), riding renderQuestRow's existing lfmHit lookup so it refreshes with the 30s tick. Verified headless (jsdom, 60 checks, ALL PASS): R4 → orange bars + "☠ R4" tag + the Saltmarsh quest row glowing orange when opened; R9 → red everything, orange gone; R2 → neutral tag only, no glow; CSS wash/border/pulse assertions hold; 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.7`, all Build .1–.6 behaviors intact. Park retention: kept `sooks-saga-scroll-07052026-7.html` + `sooks-saga-scroll-07052026-6.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-5.html` (removed 2026-07-05). Prior build 07052026.6 (superseded) — reaper-tier LFM highlights + 30-second live refresh; no schema change (still v12). (1) `refreshLfmIndex` parses comment-declared reaper skulls (R1–R10 via `\br\s?(10|[1-9])\b`; DDO's LFM difficulty field is just "Reaper" so comments are the only skull source — live-confirmed with "lamor r1" / "Eveningstar stuff on R1"; the declared value is trusted regardless of the difficulty flag) and keeps the highest per quest as `skulls`, folded into the repaint signature. Container progress bars tint by the highest declared tier among their quests: R4–R7 = brilliant orange `.lfm-mid`, R8–R10 = brilliant red `.lfm-high` (pulsing `lfmHighPulse`, behind prefers-reduced-motion), R1–R3/undeclared = no tint; declared after has-progress/all-done so the hotter glow wins; 🔥 badges + tooltips show "· R<n>". Cross-tier name matching (one LFM lights both Saltmarsh tiers) is by design. (2) 30-second live refresh: `LIVE_REFRESH_MS` setInterval + Page Visibility gating (document.hidden skips; visibilitychange fires an instant catch-up), each tick force-expires `_ddoCache`/`_guildCache`/the LFM window and re-renders medallions, guild corner, LFM index; signature guard prevents idle DOM churn; polling chosen because DDO Audit has no push channel. Verified headless (jsdom, 52 checks, ALL PASS): "r4" → skulls=4 + orange only, "R9" → red only, "r2" → no tint, CSS + reduced-motion assertions hold, manual tick force-refetches LFM/character/guild data, 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.6`, all Build .1–.5 behaviors intact. Park retention: kept `sooks-saga-scroll-07052026-6.html` + `sooks-saga-scroll-07052026-5.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-4.html` (removed 2026-07-05). Prior build 07052026.5 (superseded) — "Anonymous" roster label + brilliant progress bars; no schema change (still v12). (1) Anonymous guildmates: prompted by guildmate "Henk" (Shadowdale, ⟨Order of the Silver Dragons⟩) appearing as "?" — DDO Audit blanks a member's `name` while their in-game Anonymous flag is on (confirmed live; the single-character endpoint returns 403 for anonymous characters). `_summarizeGuild` now carries `anon` (`is_anonymous === true` OR blank name) and the roster renders those rows as a faded-italic "Anonymous" (`.gm-anon`, explanatory tooltip) with level/class kept (not redacted by the API); the `"?"` fallback is gone. (2) Brilliant progress bars: `has-progress` on `.saga-progress-bar` when `done > 0` — gold border, soft outer glow, richer green→gold fill, slow illuminated sheen across the filled portion (`@keyframes progressSheen`, gated behind `prefers-reduced-motion: no-preference`); `all-done` when `done >= total` glows warmest with a fully gold-tipped fill; empty bars keep the quiet parchment look. Verified headless (jsdom, 44 checks, ALL PASS): 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.5`, anonymous fixture renders "Anonymous" + tooltip + `Monk · Lvl 7` meta with no "?" left in source, fresh state has zero brilliant bars, marking one quest complete lights exactly that bar (others quiet), gold/glow + reduced-motion CSS assertions hold, and all Build .1–.4 behaviors still pass. Park retention: kept `sooks-saga-scroll-07052026-5.html` + `sooks-saga-scroll-07052026-4.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-3.html` (removed 2026-07-05). Prior build 07052026.4 (superseded) — the guild card folds up; no schema change (still v12). Clicking the guild-name header (or Enter/Space — `role="button"`, `tabindex="0"`) toggles the roster: folded, the card is just the header bar — caret ▸ (▾ when open), guild name + online count still visible, no `<tbody>` rendered, and quest/area ref resolution skipped while folded. Fold state persists across reloads in the META localStorage key `sooksSagaScrollMeta` (outside `state`; Export payload untouched) via new merge-style `_readAppMeta()`/`_writeAppMeta()` helpers, and the Export handler's `lastExportedAt` write was converted from a raw overwrite to `_writeAppMeta` so the fold flag survives an export (previously the export would have clobbered the meta object). New CSS `th.gm-fold` (pointer + hover brightness) and `.gm-caret`. Verified headless (jsdom, 34 checks, ALL PASS): 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.4`, click folds → no tbody + ▸ caret + header intact, click reopens → both member cells back, fold and unfold each persist to the meta key, meta writes merge (guildFolded survives a lastExportedAt write; exactly one raw setItem on the meta key remains, inside `_writeAppMeta`), and every roster behavior from Builds .1–.3 still passes (⚔ quest / 📍 zone / uniform row thickness / legibility sizes / overflow safety). Park retention: kept `sooks-saga-scroll-07052026-4.html` + `sooks-saga-scroll-07052026-3.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-2.html` (removed 2026-07-05). Prior build 07052026.3 (superseded) — guild-box legibility pass, CSS only, no schema change (still v12). The roster text was EB Garamond at 0.7–0.74rem in faded ink with italics — hard to read at the card's size, per user feedback. Body text 0.74 → 0.85rem; class/level meta and quest/zone lines 0.7 → 0.8rem; `--ink-faded` replaced with full-strength `--ink` on both member lines (only the "No guildmates online" note stays faded); italics dropped from the quest/zone line; Cinzel header 0.7 → 0.76rem; line-heights (1.25/1.3) and row padding (3 → 4px) opened slightly; `.gm-quest` reserved min-height retuned to `calc(0.8rem * 1.3)` preserving Build .2's uniform row thickness. No markup or JS change; all Build .1 overflow-safety rules retained. Verified headless (jsdom, 25 checks, ALL PASS): 0 runtime errors, self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.3`, all prior roster behaviors intact (⚔ quest / 📍 zone / uniform thickness / class-first meta), five new legibility assertions hold. Park retention: kept `sooks-saga-scroll-07052026-3.html` + `sooks-saga-scroll-07052026-2.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); pruned `sooks-saga-scroll-07052026-1.html` (removed 2026-07-05). Prior build 07052026.2 (superseded) — two user-requested refinements to the Build .1 two-line guild roster, no schema change (still v12). (1) Uniform row thickness: the second line now ALWAYS renders and reserves one line of height (`min-height: calc(0.7rem * 1.25)` on `.gm-quest`), so a member with no resolvable location gets a blank-but-reserved line instead of a thinner row. (2) Zone display: when a member is NOT inside a quest instance, line 2 shows their zone — public spaces included — as "📍 <zone>" (class `.gm-zone`, tooltip "Current zone"), resolved from the areas ref table via `getAreasMap()` (same 7-day `sooksSagaScrollRef` cache); inside a quest the line stays "⚔ <quest>" without `.gm-zone`. `renderGuildCorner` resolves both ref tables in one `Promise.all` with a stale-guard after the await. Verified headless (jsdom, 21 checks, ALL PASS): 0 runtime errors, data self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.2`, ⚔ for the in-quest member (no `.gm-zone`), "📍 The Marketplace" with `.gm-zone` for the public-area member, both cells carry a line-2 div, min-height present, and every Build .1 overflow-safety assertion still holds (table-layout fixed, no nowrap, overflow-wrap ×3, flex-wrap). Park retention: kept `sooks-saga-scroll-07052026-2.html` + `sooks-saga-scroll-07052026-1.html` + cross-day anchor `sooks-saga-scroll-06302026-1.html` (3 files); nothing pruned this park. Prior build 07052026.1 (superseded) — guild-roster rows are now TWO stacked lines per online member: line 1 = bold name + class-first heroic meta (`Fighter/Monk · Lvl 34`; Epic/Legendary/Destiny pseudo-classes still dropped), line 2 = the quest that member is currently running (⚔-prefixed italic `.gm-quest`), rendered only when their `location_id` resolves to a quest instance. Powered by a new quest-by-area index: the quests reference cache (`sooksSagaScrollRef`, 7-day TTL, outside `state`) now also stores each quest's `area_id` (slim key `area`); a shape guard treats older caches lacking the key as stale and refetches once. New `_setQuestMaps()` / `_questAreaMap` / `getQuestAreaMap()`; `_summarizeGuild` keeps each online member's `location_id`; `renderGuildCorner` resolves the index (with a stale-guard after the await) and renders one full-width `.gm-cell` per member — the table is single-column now (header th lost its colspan). Fit guarantee: `table-layout: fixed`, zero `white-space: nowrap` in the guild CSS, `overflow-wrap: anywhere` on name/meta/quest, and `.gm-line1` flex-wraps so the meta drops under a long name — nothing can run off the card's right edge. No schema change (still v12); Export/Import untouched. Verified headless (jsdom, 17 checks, ALL PASS): 0 runtime errors, data self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), build stamp `07052026.1`, header "⟨The Zhentarim⟩ · 2 online" single-column, self + offline excluded, quest line present for the in-quest member and absent for the public-area member, stale-shape cache → exactly 1 refetch and rewritten rows carry `area`, all four overflow-safety CSS assertions hold. Park retention: kept `sooks-saga-scroll-07052026-1.html` + `sooks-saga-scroll-06302026-1.html` (the latter doubles as the cross-day rollback anchor, so the retain set is 2 files); pruned `sooks-saga-scroll-06222026-4.html` (removed 2026-07-05). Prior build 06302026.1 (superseded) — CSS-only fix: the online-guildmates roster table no longer clips member names at the right edge. `.guild-table` now uses `table-layout: fixed`, and `.gm-name` (width 52%) + `.gm-meta` switched from `white-space: nowrap` to `white-space: normal` with `overflow-wrap: anywhere` / `word-break: break-word`, so long names and multiclass metas (e.g. `Lvl 34 Fighter/Monk`) wrap inside the fixed 200px card instead of overrunning the panel's `overflow-x: hidden`. No markup or JS change — `renderGuildCorner()` untouched; no schema change (still v12). Verified headless (jsdom): boots with 0 JS errors, build stamp `06302026.1`, table-layout + both cells wrapping confirmed present. Park retention: kept `sooks-saga-scroll-06302026-1.html` + `sooks-saga-scroll-06222026-4.html` (the latter doubles as the cross-day rollback anchor, so the retain set is 2 files); pruned `sooks-saga-scroll-06222026-3.html` and `sooks-saga-scroll-06212026-8.html` (removed 2026-06-30). Prior build 06222026.4 — the contextual level-relative "Show" now reveals BOTH saga and non-saga content for the character's level band, no schema change (still v12). The level-aware chip used to filter to a single tier; it now shows the whole band — saga tier + non-saga sibling: Heroic + Non-Saga Heroic (L1–19), Epic + Non-Saga Epic (L20–29), Legendary + Non-Saga Legendary (L30+). Implemented as a `band:<tier>` value of `state.tierFilter`: new `TIER_BANDS` map + `bandTiers()` / `tierFilterActiveSet()` helpers; the saga filter shows every band member; both band buttons in the segmented tier control read as active; `VALID_TIER_FILTERS` accepts the three band values so a persisted contextual show survives reload. The chip reads "suggested: <Tier> + Non-Saga <Tier>" and its button "Show <Tier> (saga + non-saga)"; it still folds up all sagas (Build .3). A single-tier button click afterward still narrows to exactly that tier. Verified headless (jsdom): `bandTiers("band:Epic")` → `["Non-Saga Epic","Epic"]`; a level-25 character's "Show Epic (saga + non-saga)" sets `tierFilter="band:Epic"`, visible cards are exactly Epic + Non-Saga Epic, both those buttons active, all sagas folded (0 open), then a single Epic-button click narrows to just Epic; data self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), 0 runtime errors, build stamp `06222026.4`. Park retention: kept `sooks-saga-scroll-06222026-4.html` + same-day previous `sooks-saga-scroll-06222026-3.html` + cross-day anchor `sooks-saga-scroll-06212026-8.html` (3 files); pruned `sooks-saga-scroll-06222026-2.html` (removed 2026-06-22). Prior build 06222026.3 (superseded) — two UX refinements, no schema change (still v12). (1) The upper-right floating cluster (`.data-corner`) is no longer one box; it's a transparent wrapper stacking TWO clearly-distinct cards with a gap: a CHARACTERS card (`.corner-panel.char-panel` — small "Characters" caption + the chip list `#charCorner` + the Export/Import buttons, so the data actions read as belonging to the roster) and a separate GUILD card (`.corner-panel.guild-panel`, formerly `#guildCorner`, holding the online-guildmates table headed by the guild name). Both cards are a fixed 200px wide so they align; the guild card hides when empty; the mobile media query's padding moved from `.data-corner` to `.corner-panel`. New CSS `.corner-panel`/`.char-panel`/`.corner-cap`/`.guild-panel` replace `.guild-corner`. (2) The level-aware "Show <Tier>" button now FOLDS UP all sagas — it sets Expand-all off and clears the `expanded`/`collapsed` sets — so showing quests for the character's relative level gives a tidy collapsed overview to expand selectively. Verified headless (jsdom): the cluster renders exactly two cards in order Characters → Guild, Export + Import live inside the Characters card, the guild table is a separate card with header "⟨The Zhentarim⟩ · 2 online", and clicking "Show Epic" for a level-25 character sets `tierFilter=Epic` with Expand-all off, both expand sets empty, and 0 open saga cards; data self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), 0 runtime errors, build stamp `06222026.3`. Park retention: kept `sooks-saga-scroll-06222026-3.html` + same-day previous `sooks-saga-scroll-06222026-2.html` + cross-day anchor `sooks-saga-scroll-06212026-8.html` (3 files); pruned `sooks-saga-scroll-06222026-1.html` (removed 2026-06-22). Prior build 06222026.2 (superseded) — guildmates-online relocated from the medallion into a floating roster TABLE in the upper-right cluster (`.data-corner` → new `#guildCorner`, directly under the `#charCorner` character-chip list, above Export/Import). The table HEADER is the active character's guild name — "⟨<Guild>⟩ · N online" — and each body row is an online guildmate's name (`.gm-name`) + "Lvl L Class" (`.gm-meta`, heroic multiclass like "Lvl 34 Fighter/Monk"); self and offline members excluded. Sticky header, `max-height: 40vh` internal scroll, 180px column matching the chips; hidden (`:empty`) until the active character has a server set and DDO Audit reports a guild. New `renderGuildCorner()` (called from `renderAll` + `updateCharStats`), reusing the Build .1 live path (`fetchCharacter`→`guild_name`, `fetchGuildmates`→roster; array + object `data` shapes handled), best-effort/async with stale guards. The Build .1 per-medallion "🛡 N guildmates online" line is removed. No schema change (still v12); new CSS `.guild-corner`/`.guild-table`/`.gm-name`/`.gm-meta`/`.gm-empty`. Verified headless (jsdom): table renders under `#charCorner` inside `.data-corner` with header "⟨The Zhentarim⟩ · 2 online", rows Kaeithra/Sparkeee (self + offline excluded), sticky header, old medallion `.guildmates` line gone (0), hides when the active char has no server, data self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), 0 runtime errors, build stamp `06222026.2`. Park retention: kept `sooks-saga-scroll-06222026-2.html` + same-day previous `sooks-saga-scroll-06222026-1.html` + cross-day anchor `sooks-saga-scroll-06212026-8.html` (3 files); nothing pruned. Prior build 06222026.1 (superseded) — five DDO Audit live-data features (no schema change, still v12), all best-effort / offline-first / CORS-safe and keyed off the active character's server, with NO new persisted state: (1) live location resolves a char's numeric `location_id` via `GET /v1/areas` → "· 📍 <area>" on the medallion, with a conservative name/pack/region match that highlights the matching saga card (`saga-now-playing`) + a jump-to banner; (2) level-aware tier-suggestion chip (reuses persisted `characterLevels`: 1–19 Heroic / 20–29 Epic / 30+ Legendary, with a "Show <Tier>" button + to-next math); (3) last-seen on offline medallions (`last_update`→`relativeTime()`, e.g. "Offline · seen 3 days ago"); (4) guildmates-online via `GET /v1/characters/by-server-and-guild-name/{server}/{guild}` → "🛡 N guildmates online" excluding self; (5) live LFM badges — `GET /v1/lfms/{server}` + `GET /v1/quests` (quest_id→name, skip 0) build an index matched by `normalizeQuestName()`, rendering "🔥 N LFM" on quest rows with a 60s-window + change-signature guard so async re-renders can't loop (verified 4 renders → 1 fetch). Reference tables cached OUTSIDE `state` in localStorage key `sooksSagaScrollRef` (7-day TTL); live LFM/guild cached ~60s in memory. Only the new servers (Cormyr/Moonsea/Shadowdale/Thrane) carry live data; older servers return empty and the features stay inert. New CSS: `.char-loc`, `.guildmates`, `.quest-lfm`, `.tier-suggest`, `.loc-banner`, `.saga-now-playing`. API shapes confirmed against the DDO Audit OpenAPI (`api.ddoaudit.com/docs/openapi.json`) + live Shadowdale payloads. Verified headless (jsdom): page boots with the network down with 0 runtime errors, data self-check unchanged (✓ 62 containers / 517 quests / 100% detailed), schema still v12 and Export→Import round-trips with no new fields, location resolves ("Tempest's Spine"), tier mapping + to-next math correct, last-seen renders, guildmates excludes self, LFM badge only on matched quests, build stamp `06222026.1`. Park retention: kept `sooks-saga-scroll-06222026-1.html` + cross-day anchor `sooks-saga-scroll-06212026-8.html` (also the 2nd-most-recent, so the retain set is 2 files); pruned `sooks-saga-scroll-06212026-7.html` and `sooks-saga-scroll-06152026-2.html` (removed 2026-06-22). Prior build 06212026.8 (superseded) — medallion meta shows heroic classes only (no schema change, v12). DDO Audit lists "Epic"/"Legendary" as pseudo-class entries in the classes array (e.g. Fighter 20 / Epic 10 / Legendary 2); `describeCharacter` now drops Epic/Legendary/Destiny so the class breakdown is heroic-only — the aggregate total level already conveys the epic/legendary span. Total level unchanged; one-line JS filter. Verified headless (jsdom, 7 checks): Epic/Legendary removed, single/multi-class heroic preserved, total level intact (e.g. `Shadowdale · Lvl 32 · Human Fighter 20 · ⟨Guild⟩`). Park retention: kept `sooks-saga-scroll-06212026-8.html` + same-day previous `sooks-saga-scroll-06212026-7.html` + cross-day anchor `sooks-saga-scroll-06152026-2.html` (3 files); pruned `sooks-saga-scroll-06212026-6.html` (removed 2026-06-21). Prior build 06212026.7 — medallion polish + offline level persistence (schema v11 → v12). (1) Live line leads with the aggregate/total level (`total_level` or class-sum). (2) Every medallion is fixed-size (width 200px; name clamped to 1 line, stats to a fixed 2-line height with ellipsis) so metadata can't spill and a no-metadata medallion matches one with metadata. (3) No server set now shows "Missing server" (was: no line) instead of "Offline mode". (4) New additive `characterLevels` map (v12) captures the aggregate level from each lookup — online or last-known-from-DB while offline — and persists it for future offline level-based features; mirrors `characterServers`, synced on add/rename/delete, round-trips via Export. Verified headless (jsdom, 15 checks): schema v12, aggregate from total or class-sum, level persisted on online+offline lookups, "Missing server" rendered, all medallions carry a stats line for uniform sizing. Park retention: kept `sooks-saga-scroll-06212026-7.html` + same-day previous `sooks-saga-scroll-06212026-6.html` + cross-day anchor `sooks-saga-scroll-06152026-2.html` (3 files); pruned `sooks-saga-scroll-06212026-5.html` (removed 2026-06-21). Prior build 06212026.6 — DDO Audit endpoint corrected (no schema change, v11). Build .5's self-diagnostic proved `api.ddoaudit.com` is reachable + CORS-open (a real HTTP 404, which CORS blocks can't produce) — the path was wrong. Confirmed from the backend source (`Clemeit/ddo-audit-service`, `sanic/endpoints/characters.py`): the `characters` blueprint is `version=1`, so the base is `/v1`, with a direct single-character endpoint `GET /v1/characters/<server>/<name>` → `{data, source}` (`source:"database"` = offline), `404` not-found, `403` anonymous. Rewrote the lookup to call it directly (proper- then lower-case server; no more list-scan) and render the real snake_case model (`total_level`, `classes[{name,level}]`, `race`, `guild_name`, `server_name`, `is_online`, `is_anonymous`); added a distinct "Anonymous" state; kept the self-diagnosing "Offline mode" tooltip. JS only. Verified headless (jsdom, 12 checks): exact `/v1/characters/{server}/{name}` URL, online-200 renders the full line, and offline/404/403/CORS each map to the right state incl. e2e medallion. Park retention: kept `sooks-saga-scroll-06212026-6.html` + same-day previous `sooks-saga-scroll-06212026-5.html` + cross-day anchor `sooks-saga-scroll-06152026-2.html` (3 files); pruned `sooks-saga-scroll-06212026-4.html` (removed 2026-06-21). Prior build 06212026.5 — DDO Audit lookup fix + self-diagnosis (no schema change, v11). A logged-in character still showed "Offline mode" because Build .4 only queried `/players/{server}`; the rewritten backend serves live characters at `api.ddoaudit.com/characters/{server}` (the `/who` page). Made `/characters/{server}` the primary URL (then `/players`, then legacy `playeraudit.com`), broadened the parser to accept `characters` / `data.characters` shapes, and made "Offline mode" self-diagnose — hover or console shows each URL's outcome (`HTTP 404`, `blocked by browser (CORS)`, `reached but shape not recognized`, `reached N online but name not found`) to distinguish wrong-endpoint vs CORS-block vs name-typo. JS only. Verified headless (jsdom, 13 checks): `/characters` shape with classes/guild/location renders the full live line; HTTP-404 / CORS-throw / no-match each yield the right tooltip. Park retention: kept `sooks-saga-scroll-06212026-5.html` + same-day previous `sooks-saga-scroll-06212026-4.html` + cross-day anchor `sooks-saga-scroll-06152026-2.html` (3 files); pruned `sooks-saga-scroll-06212026-3.html` (removed 2026-06-21). Prior build 06212026.4 — DDO Audit live character status (schema v10 → v11). Optional, non-blocking: new additive per-character `characterServers` map (mirrors `characterThemes`; round-trips via whole-state Export). Pick a character's DDO server in Manage and the medallion shows a live `Server · Lvl N · Classes · ⟨Guild⟩ · Location` line from DDO Audit when online, else **"Offline mode"** (offline / name mismatch / API unreachable); no server set = no line, medallion unchanged. Only network call in the app; queries a server's live player list (`api.ddoaudit.com`, legacy `playeraudit.com` fallback) and matches the name — no "find by name" endpoint exists (per the official DDO Audit Discord bot source). Endpoint/parse isolated in `DDO_AUDIT.urls` / `parsePlayers`. Caveat: CORS from a `file://` page is unverified — if blocked, the line always reads "Offline mode" (safe, non-breaking). Verified headless (jsdom, 26 checks): schema v11, `characterServers` survives ensureShape + Export round-trip, parser handles all known shapes, stubbed live match renders the metadata line, fetch-reject and no-match both fall back to "Offline mode", and the Manage dropdown + medallion wire up end-to-end. Park retention: kept `sooks-saga-scroll-06212026-4.html` + same-day previous `sooks-saga-scroll-06212026-3.html` + cross-day anchor `sooks-saga-scroll-06152026-2.html` (3 files); pruned `sooks-saga-scroll-06212026-2.html` (removed 2026-06-21). Prior build 06212026.3 — tier-filter button order: swapped the Legendary and Non-Saga Legendary filter buttons so every pair reads non-saga-first (Non-Saga Heroic → Heroic, Non-Saga Epic → Epic, Non-Saga Legendary → Legendary); previously Legendary preceded Non-Saga Legendary, the only saga-first pair. Filter buttons only — section render/group order and all data unchanged; search box intact. No JS/schema change (still v10). Verified (jsdom): order is `Non-Saga Heroic | Heroic | Non-Saga Epic | Epic | Non-Saga Legendary | Legendary | all`, search input present, single active tier, build stamp correct. Park retention: kept `sooks-saga-scroll-06212026-3.html` + same-day previous `sooks-saga-scroll-06212026-2.html` + cross-day anchor `sooks-saga-scroll-06152026-2.html` (3 files); pruned `sooks-saga-scroll-06212026-1.html` (removed 2026-06-21). Prior build 06212026.2 — tier-filter spill guard (CSS only): the flat tier buttons from .1 could shrink below their label width in the flex row (default flex-shrink) and, with `white-space: nowrap` and no overflow handling, let text bleed past the box borders on a narrow window. Pinned each button to `flex: 0 0 auto` (never shrinks) + inline-flex centering so the box always hugs its text; `.tier-segment.bar` now scrolls horizontally (`overflow-x: auto`) if the row can't fit instead of squeezing the boxes. No JS/data/schema change (still v10). Verified structurally (jsdom): 7 boxes, single active tier, build stamp, fresh-load empty state byte-identical to .1; spill-prevention rests on the deterministic CSS (a `flex: 0 0 auto` + `nowrap` + `border-box` box is exactly label-width, so text can't exceed it). Park retention: kept `sooks-saga-scroll-06212026-2.html` + same-day previous `sooks-saga-scroll-06212026-1.html` + cross-day anchor `sooks-saga-scroll-06152026-2.html` (3 files); nothing pruned. Prior build 06212026.1 — tier filter restyled (CSS/markup only): the page-header tier filter went from a 520px two-row wax-tablet grid (`.tier-segment.grid`, wrapping 4-over-3) to a tidy single row of modern, flat, clearly-boxed buttons (`.tier-segment.bar`) that line up to the search-box height — each tier its own crisp 1px-bordered parchment box, active tier a solid wax-red fill, "All" flattened from the old rounded gold pill and set apart by an 8px gap. Markup: container class `grid`→`bar`, buttons dropped the unused `grid-btn` class. No JS touched (click/active logic keys off `.seg-btn` + `data-tier` + `.active`), no data or schema change (still v10), Import/Export contract untouched. Verified headless (jsdom): 0 new JS errors, 7 `.seg-btn` boxes, single active tier, clicking a tier flips the active box, and the fresh-load empty state is byte-identical to Build 06152026.2. Park retention: kept `sooks-saga-scroll-06212026-1.html` + cross-day anchor `sooks-saga-scroll-06152026-2.html` (also the 2nd-most-recent, so the retain set is 2 files); pruned `sooks-saga-scroll-06152026-1.html` and `sooks-saga-scroll-06082026-5.html` (removed 2026-06-21). Prior build 06152026.2 — three story-arc bestower fills from a project-wide audit (62 containers, ~95 arcs). Heroic Harbor chains: `ns-h-harbinger` "Harbinger of Madness" giver set to **Tessa Aster** (per-quest: Parker Pipewort, Felicity Mallow, Hector Hyssop — source ddowiki Harbinger of Madness); `ns-h-web-of-chaos` "Web of Chaos (chain — Lords of Dust)" giver set to **Inquisitor Lightbringer** (Harbor, north end bridge north of The Waterworks — source ddowiki Inquisitor Lightbringer + The Lords of Dust). Epic Eberron: `e-eberron` "The Web of Chaos" arc reuses the same 3 quests; carried Inquisitor Lightbringer through after confirming via the End of Eberron (Epic) saga page that the saga-level Gatekeepers grant the saga reward only, not the chain — no distinct epic-context chain bestower exists. Audit also found 25 intentional nulls (Side Quests, Wilderness, Intro singletons) requiring no action, and 2 likely-intentional Gianthold collection arcs that weren't exhaustively re-verified this build. Data-only, schema unchanged (v10), Import/Export contract untouched. Park retention: kept `sooks-saga-scroll-06152026-2.html` + `sooks-saga-scroll-06152026-1.html` + cross-day anchor `sooks-saga-scroll-06082026-5.html`; pruned `sooks-saga-scroll-06082026-4.html` (removed 2026-06-15). Prior build 06152026.1 — Cydonie set as the epic Three-Barrel Cove arc bestower. Prior build 06082026.5 — Harbor polish: (1) Harbor Standalones sort by heroic level; (2) tier filter restyled as a uniform equal-size grid that wraps to two rows; (3) Heroic / Non-Saga Heroic flipped in the filter (non-saga first); (4) Reaper button removed for the 5 wiki-confirmed solo-only / "Non-reaper" Harbor quests (Protect Baudry's Interests, Stop Hazadill's Shipment, Arachnophobia, Home Sweet Sewer, The Miller's Debt) — sourced from the DDO wiki Reaper-difficulty page, and excluded from each container's Reaper count so badges can still seal. Prior build 06082026.4 — (1) per-card collapse now works even when "Expand all" is on (new `collapsed` Set; header click toggles the individual card so it can be tucked away); (2) the single "The Harbor" container was split into FIVE per-pack Non-Saga Heroic containers (Baudry Cartamon, The Lost Seekers, Harbinger of Madness, Web of Chaos, Harbor Standalones), matching the NSL/NSE one-card-per-pack model. Container IDs changed, so progress saved under the old `ns-h-harbor` id (builds .2/.3) does not carry over. Self-check: 0 errors, 62 containers / 517 quests, all detailed. Prior build 06082026.3 — fix: the Non-Saga Heroic filter showed nothing because `renderSagaList()` had a second hardcoded tier order (`groupOrder`) that omitted the new tier; added it. Prior build 06082026.2 — new sixth tier **Non-Saga Heroic** holding "The Harbor (Stormreach)": 37 wiki-listed Harbor quests (L2–17) in 5 arcs (Baudry Cartamon, The Lost Seekers, Harbinger of Madness, Web of Chaos, + standalones), each with wiki-sourced story/monsters/traps/tips/giver/patron; Irestone Inlet filed under h-pirates, Lost at Sea already under h-masterminds. Schema bumped v9 → v10 (additive tier value). Self-check: 0 errors, 58 containers / 517 quests, all detailed. Prior build 06082026.1 — Druid's Deep arc bestower filled: ddowiki confirms **Anazar Narroun** bestows "The Druid's Deep"; set as the arc giver on both the Heroic and Epic Perils of Cormyr sagas (was `giver: null`). Prior build 06072026.10 — shared-quest (↔) tag now has a working custom tooltip on hover and tap (replaces the unreliable native title). Prior build 06072026.9 — restyled the tier filter tabs + "All" pill as embossed wax-seal tablets (gold trim, raised highlight, glowing wax-red active state). Prior build 06072026.8 — removed the "Overall Quest Progress" bar from the header rollup and restored the per-saga card bars (corrects .7, which removed the wrong element). Prior build 06072026.7 — removed per-saga progress bars (SUPERSEDED — wrong element). Prior build 06072026.6 — filled 34 missing Non-Saga Legendary per-quest givers (incl. the whole Free-to-Play Standalone container) from the wiki; 0 NSL quests now lack a giver. Prior build 06072026.5 — per-quest patron exceptions: 6 quests in mixed-patron containers now show their true wiki patron ("The Free Agents") via a conditional Path-block row. Prior build 06072026.4 — corrected all 43 Non-Saga Legendary per-quest givers to the wiki bestowers (story-arc givers retained in storyArcs). Prior build 06072026.3 — full 479-quest wiki accuracy audit; fixed Disciples of Rage patron (The Harpers → House Jorasco). Prior build 06072026.2 — Wave 8 doorLocation + NPC location backfill (17/86 NSL doors sourced, +10 NPC locations, Hworrl spelling fix). Prior same-day build 06072026.1 added the backup nudge + schema self-check and retired CONTEXT.md (README is now the single source of truth). See the build notes in the **Files** section above. Older ladder entries preserved below.) (Build 05312026.1 — **Completed-quest rollup.** Quests marked with the master Quest Completed flag (`sagaDone`) are now pulled out of each saga's main quest list and tucked into a collapsible "✓ Completed (N)" drawer at the bottom of that saga's quest list, to reduce on-screen clutter. New `renderQuestListBody()` splits active vs. completed; `renderStoryArc()` now filters completed quests (a fully-completed arc renders nothing); new `renderCompletedRollup()` builds the drawer by reusing `renderQuestRow()` so tucked quests keep all controls. Drawer is collapsed by default on every load via an in-memory `rollupOpen` Set — UI-only, **not** persisted, so **no SCHEMA_VERSION bump** (still v9, Import/Export contract untouched). Trigger is the Quest Completed master mark only; Reaper/Skip alone don't tuck a quest. CSS added: `.completed-rollup*`, `.all-complete-note`. Verified via jsdom: 52 cards render with no JS errors, flat + story-arc sagas both tuck completed quests, drawer collapsed-by-default and click-toggles open. Park retention: kept `sooks-saga-scroll-05312026-1.html` + cross-day anchor `sooks-saga-scroll-05282026-3.html`; pruned `sooks-saga-scroll-05282026-2.html` and `sooks-saga-scroll-05272026-12.html`.)

— Build 05282026.3 (superseded): Mists of Ravenloft quest order corrected to match the canonical wiki saga page. Both `h-mist` and `l-mists` now lead "In the Shadow of the Castle" with **Death House** before Fresh-baked Dreams, and lead "The Light of the Land" with **Ravens' Bane** before the rest. Other 50 containers swept: only the two Mists saga pages publish a flat ordered list on the wiki — every other saga page defers to the pack page, and the build already mirrors pack-page arc order, so no other reorderings were needed. Filename convention switched to full kebab-case (no underscores anywhere) — current files: `sooks-saga-scroll-05282026-3.html`, `sooks-saga-scroll-05282026-2.html`, `sooks-saga-scroll-05272026-12.html`. Park retention kept _2 + cross-day anchor _05272026-12; pruned _05282026-1.

— Build 05282026.2 (superseded): Removed the small accent dot from `.char-action-btn::before` and the `.add` / `.manage` / empty-state ::before override rules. Add Character + Manage buttons now show their text label only — cleaner alignment, less visual noise next to the medallions. Current file: `sooks-saga-scroll-05282026_2.html` (since renamed `-2.html`). Park retention kept _05282026_1 + cross-day _05272026_12; pruned _05262026_6.

— Build 05282026.1 (superseded): Wave 5 — NPC location backfill. Grew `NPC_LOCATIONS` from 91 → 217 entries: 49 lifted from saga-context strings already in `SAGAS[]` but never duplicated to the dictionary, plus 126 fetched directly from each NPC's ddowiki.com page via Chrome MCP using the `Location:` infobox. About 18 NPCs remain unsourced (wiki page either does not exist or carries no `Location:` field) and fall through to the renderer's "location not yet sourced — see wiki" placeholder. Source rule: every entry is a verbatim quote of the wiki Location string, ddowiki spelling preserved. Current file: `sooks-saga-scroll-05282026_1.html` (since pruned).

— Build 05272026.11 (superseded): Three changes. (1) Wave 4 correction — quest giver vs dungeon door. Build .10 wrongly populated doorLocation for all 86 non-saga Legendary quests from the wiki "Quest acquired in:" infobox, but that field is the NPC's public area, not necessarily the dungeon door. The two are NOT always the same. Cleared all 86 entries; the "Dungeon door" row now correctly shows "Not yet sourced — see wiki" until each quest's wiki Overview is explicitly searched for door/portal/entrance text. The "Talk to" row (NPC location) is unaffected — still contextually given via NPC_LOCATIONS. (2) Survival field is no longer a wall of text — new formatSurvivalList() splits at sentence boundaries and renders as bullets with a fleur (❦) glyph in wax-red, EB Garamond body, dotted dividers. Short notes stay as a single paragraph. (3) Puzzle layout restructured — block layout with wax-red left rule, Type label in Cinzel uppercase, Solution bumped to 1.18rem EB Garamond bold wax-red, Notes italic at 1rem. Parchment background + 12px gap between puzzles so multi-puzzle quests read clearly. Park retention: kept `_05262026_6` cross-day anchor + `_05272026_10` + `_05272026_11`; pruned `_05272026_9`. Current file: `sooks-saga-scroll-05272026_11.html`.

— Build 05272026.10 (superseded): Wave 4 dungeon-door + Path-block trim. Fetched the wiki "Quest acquired in:" infobox field for every one of the 86 non-saga Legendary quests via Chrome MCP (15 batches of 6 quests each). 86/86 quests now carry a `doorLocation` string sourced verbatim from the wiki. Path block dropped Travel-to and Takes-place-in rows (in both renderPathBlock and the Path-edit template buildPathTemplate); fields stay in the durable schema for existing saga entries, only the renderer changes. Path now shows 4 rows: Saga · Pack required · Talk to · Dungeon door. Also refreshed CONTEXT.md — it was from May 26, predating Wave 1/2/3, schema v9, the non-saga tiers, and all the tier-filter / non-saga UX changes. New CONTEXT.md is a slim quick-reference matching current state; README.md remains the full history. Park retention: kept `_05262026_6` cross-day anchor + `_05272026_9` + `_05272026_10`; pruned `_05272026_8`. Current file: `sooks-saga-scroll-05272026_10.html`.

— Build 05272026.9 (superseded): Wave 3 puzzles backfill for non-saga Legendary quests. Scanned all 86 non-saga Legendary quests' tips for puzzle indicators, drilled into each flagged page via Chrome MCP, extracted structured puzzle data verbatim from each wiki Tips/Walkthrough section. Added `puzzles: [{type, solution, notes}]` arrays to 20 quests across 15 containers — 26 puzzle entries total. Schema matches existing saga-quest puzzles shape, so the renderer surfaces them in the quest details card without any UI change. Notable solutions captured: Seizing the Dawn's 5-layout button sequence table, Burning Down the House's two wheel-puzzle solutions, Multitude of Menace's 4-rune-light combination, ToEE: Air Node's 3-rune Dragon's Hoard puzzle, The Archons' Trial's trap-puzzle warning, Members Only's kitchen-lookup rune combination, and White Plume Mountain's Gynosphinx riddle. Park retention: kept `_05262026_6` cross-day anchor + `_05272026_8` + `_05272026_9`; pruned `_05272026_7`. Current file: `sooks-saga-scroll-05272026_9.html`.

— Build 05272026.8 (superseded): Tier filter redesign, take two. Build .7's overlapping chevron clip-path was clipping label points where buttons overlapped. Replaced with flat rectangular buttons separated by decorative double-chevron `❯❯` arrow glyphs in wax-red. Arrows do the directional work; buttons stay readable. Hover on a button nudges it 2px right to reinforce flow; a staggered `flowPulse` keyframe ripples the arrows L→R on a 2.4s loop. "All" reset pill stays on the far right with a 22px spacer. No data or schema changes. Park retention: kept `_05262026_6` cross-day anchor + `_05272026_7` + `_05272026_8`; pruned `_05272026_6`. Current file: `sooks-saga-scroll-05272026_8.html`.

— Build 05272026.7 (superseded): Two coupled UX changes + a data fix. (1) Tier filter restyled: the five tier buttons are now right-pointing chevron pentagons via CSS clip-path (breadcrumb-style with overlapping points), separated by a 22px gap from a standalone "All" reset pill on the far right. (2) First Time Reaper filter toggle removed entirely — the per-saga REAPER badge and the per-quest Reaper button already convey FTR state, so the filter was redundant. state.ftrOnly remains in the schema for v8/v9 export compatibility but is force-cleared on load so a stale true value can't silently hide content. (3) Wave 2.2.1 data fix: 6 of the 86 non-saga Legendary quests had thin/missing tips after Wave 2.2 (regex extractor over-matched footer nav for some, under-matched empty Overview sections for others). Re-fetched all 6 with a DOM-based section extractor (reads each h2's siblings until the next h2 — clean boundaries, no false matches). Quests fixed: Secret of the Slavers' Stockade, Dread Sea Scrolls, Beautiful Nightmares, Grim and Barett, Thrall of Duty (story pulled from page intro since the Overview h2 was empty), Desire in the Dark. All 86 non-saga Legendary quests now carry full story / tips / monsters / traps verbatim from their wiki pages. Current file: `sooks-saga-scroll-05272026_7.html`. Park retention: kept `_05262026_6` cross-day anchor + `_05272026_6` + `_05272026_7`; pruned `_05272026_5`.

— Build 05272026.6 (superseded): Bug fix — per-quest level chip on the far-right of each quest row was silently not rendering for non-saga rows because the renderer lowercased the tier name to index `QUEST_DETAILS.levels` — "Non-Saga Legendary" → "non-saga legendary" which has no levels mapping. Fixed with an explicit tier-key mapping ("Non-Saga Epic" → "epic", "Non-Saga Legendary" → "legendary") before the lookup. Wave 2.1 levels data flows through to the chip on every non-saga row now. Saga rows unaffected. Schema unchanged. Current file: `sooks-saga-scroll-05272026_6.html`. Park retention: kept `_05262026_6` cross-day anchor + `_05272026_5` + `_05272026_6`; pruned `_05272026_4`.

— Build 05272026.5 (superseded): Wave 2.2 ships — comprehensive per-quest wiki-sourced detail for all 86 non-saga Legendary quests. Every quest now carries story (Overview), monsters (full `table.monster-table` with per-monster wiki URLs), traps (Known Traps section), notes (Tips and Misc), wikiUrl, plus the levels from Wave 2.1. Fetched serially via Chrome MCP per quest page — no inference. Integrity check: 86/86 with monsters. Schema unchanged. Total file growth: 232k → 700k of QUEST_DETAILS. Current file: `sooks-saga-scroll-05272026_5.html`. Park retention: kept `_05262026_6` cross-day anchor + `_05272026_4` + `_05272026_5`; pruned `_05272026_3`.

— Build 05272026.4 (superseded): Non-saga UX cleanup. Two targeted renderer changes scoped to `saga.isNotSaga === true` only: (1) per-quest **★ SKIP ★** button hidden — non-saga doesn't track skips; the `skipDone` field, SKIP_CAP_PER_SAGA cap, and click handler stay in place so saga rows are untouched. (2) Container completion **relabeled from "Banked" to "Completed"** — header pill, body button, dialog wording all updated; `.banked` CSS class stays as the visual hook for green-wax styling. Per-quest "Complete for SAGA" button becomes "Mark as COMPLETE" / "Marked COMPLETE" on non-saga rows. "End Reward" label becomes "Tracking". Saga progress bar tooltip drops "for saga reward" suffix. BANKED → INCOMPLETE confirm dialog uses "container" instead of "saga" and skips the Skip-slots bullet on non-saga. Schema unchanged from v9 (storage shape is the same; only renderer labels flip). Current file: `sooks-saga-scroll-05272026_4.html`. Park retention: kept `_05262026_6` cross-day anchor + `_05272026_3` (Wave 2.1) + `_05272026_4`; pruned `_05272026_2`.

— Build 05272026.3 (superseded): Wave 2.1 — per-quest level reference for all 86 non-saga Legendary quests. Every quest in QUEST_DETAILS now carries `levels: {legendary, heroic}` matching the saga-quest schema, sourced verbatim from the DDO wiki Quests-by-level-and-XP Legendary table. `wikiUrl` populated per quest. Story/monsters/traps/tips remain stubbed with `monstersUnsourced: true` for the next backfill pass (Wave 2.2 — per-quest wiki page visits). Quest level chips will now render on every non-saga Legendary quest row, matching the existing saga-quest UI. Total project quest count: 454 (3 Heroic-tier sagas + 7 Epic-tier sagas + 9 Legendary-tier sagas covering 363 quests, plus 4 Phiarlan Carnival + 1 Time Flies + 86 non-saga Legendary = 91 non-saga quests). Current file: `sooks-saga-scroll-05272026_3.html`. Park retention: kept the cross-day anchor `_05262026_6` plus `_05272026_2` and `_05272026_3`; pruned `_05272026_1`.

— Build 05272026.2 (superseded): Non-saga corrections after in-game review of .1. Tier order changed to **Heroic → Non-Saga Epic → Epic → Legendary → Non-Saga Legendary** (no more bookending). Non-Saga Epic replaced: Build .1's 3 entries (Lord of Blades / Tower of Despair / The Dreaming Dark) were all raids per the DDO Wiki Raids category page; non-saga tiers explicitly do NOT track raids. Replaced with **Phiarlan Carnival** (4 quests, The Maleficent Cabal arc bestowed by Rouge — full QUEST_DETAILS with monsters/traps/tips for all 4) and **Time Flies** (Anniversary Celebration, bestower Sariyon Dran-dal — full QUEST_DETAILS). Non-Saga Legendary pruned: dropped 17 raids identified on the wiki Raids page (Chronoscope-L, Riding the Storm Out, Killing Time, Hunt or Be Hunted, Master/Lord of Blades Legendary, Fire on Thunder Peak, Temple of the Deathwyrm, Codex and the Shroud, Vision of Destruction Legendary, Defiler of the Just, Mark of Death, Too Hot to Handle, Project Nemesis, Tempest's Spine Legendary, Hound of Xoriat Legendary, Den of Vipers). Five containers became empty after raid removal and were deleted (devil-assault, artificers-extras, thunderholme, necropolis-4, masterminds-extras). Also dropped Lost at Sea (lives in l-masterminds pre-quest arc) and moved Time Flies from Legendary bucket to Non-Saga Epic. Final tally: 23 non-saga containers (2 Epic + 21 Legendary) carrying 91 quests (5 Epic L20 + 86 Legendary); total project quest count 454. Wave-1 deliverable; Wave 2 will backfill per-quest QUEST_DETAILS for the 86 Legendary quests (monsters, traps, tips per quest wiki page). Current file: `sooks-saga-scroll-05272026_2.html`. Old summary below preserved for posterity.

— Build 05272026.1 (superseded): Wave 1 of the two new non-saga tiers. Schema bumped 8→9. Adds `"Non-Saga Epic"` and `"Non-Saga Legendary"` as valid `tier` values, bookend group order (Non-Saga Epic first, then Heroic / Epic / Legendary, Non-Saga Legendary last), and 29 new SAGAS containers with `isNotSaga: true` — 3 standalone Epic L20 raids (Lord of Blades, Tower of Despair, Dreaming Dark) and 26 Legendary standalone containers covering every non-saga Legendary pack the wiki lists (Slave Lords, Attack on Stormreach, Devil Assault, Disciples of Rage, Dragonblood Prophecy, Fall of the Night Brigade, Free-to-Play singles bucket, Grip of the Hidden Hand, Hunter and Hunted, Illithid Invasion, Masterminds outliers, Peril of the Planar Eyes, Artificers Legendary raids, Thunderholme raids, Slice of Life, Tavern Tales Legendary, Devil's Gambit, Dragons' Hand, Lost Gatekeepers, Mines of Tethyamar, Necropolis Part 4, Soul Splitter, ToEE — full per-quest bestowers, Vale of Twilight, Trials of the Archons, White Plume Mountain). Tier filter row gains two extra-wide buttons (Non-Saga Epic 130px, Non-Saga Legendary 156px); the original four shrink to 80px. Saga renderer detects `isNotSaga` and replaces the "Saga Giver / Location" rows with a single italic "Not a saga" note. Pack-level chain bestowers, storyArcs, and NPC_LOCATIONS populated from DDO Wiki pack pages — no inference; per-quest detail (story, monsters, traps, puzzles) marked as "Not yet sourced — see wiki" for the next research wave. Total quest count: 353 → 470. Current file: `sooks-saga-scroll-05272026_1.html`.)

