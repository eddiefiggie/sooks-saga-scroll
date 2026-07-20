# Sook's Saga Scroll — Session-Run Workflow & Performance Ideation

**Date:** 2026-07-19 · **Subject:** custom quest ordering · autocomplete confirmation + session queue · continued CPU efficiency · **Mode:** repo-grounded · **Format:** markdown

Focused follow-up to [`2026-07-19-001-sooks-saga-scroll-feature-ideas.md`](2026-07-19-001-sooks-saga-scroll-feature-ideas.md) (the broad sweep). This run takes four user-named directions, grounds each in the actual parked build (`sooks-saga-scroll-07192026-2.html`) via three code-reading lenses, and ranks the strongest framings. Right-sized: 3-lens fleet, no web research (offline app).

## Grounding context (verified against source)
- Quest order today is **static** — from the hardcoded `SAGAS[].quests` arrays (and `arc.quests` for multi-arc sagas). Nothing per-character reorders it.
- Per-saga per-character state lives on `getSagaState(char, sagaId)` → `{quests, sagaDone, skipDone, reward, notes}` — the natural additive home for an order overlay. A global questName-keyed overlay pattern already exists (`questOverrides`).
- Autocomplete: `_autoCompleteCheck` fires on the ~15s tick, de-dups via `_acSeen`, and calls `_acApply` which **silently mutates** state (sets `sagaDone`, cascades `SHARED_QUESTS` siblings, auto-banks, writes `quests[q]="reaper"`, feeds `reaperPopQueue`). An in-memory `_acLog[]` trace already records every apply. Prefs are per-character `{on, mode}` (v14). The earlier FTR bug fix is the `wantReaper = mode==="reaper" && !NO_REAPER.has(q) && !alreadyReaper` gate.
- **Render granularity is the standing perf liability:** nearly every interaction calls `renderSagaList()`, which does `list.innerHTML=""` and rebuilds *every* visible card + re-wires all listeners + re-scans `lfmForTier`. Background polling is already correctly gated on `document.hidden` (good). The DS&A indexes from Build .1 are effective; the remaining waste is render **frequency/granularity** and per-poll rebuild passes.
- **Constraint:** stays single-file/offline, no backend; localStorage schema v14 — every idea below is additive (no destructive migration).

---

## Direction A — Custom quest ordering

**Recommended design (survivor): Sort-preset control + Manual editor, per-character per-saga, arc-aware.**
- A per-saga **order control** mirroring the existing saga-level `#groupBy <select>`: **Wiki default · By level · A→Z · Manual**. Presets are pure functions (zero storage — recomputed each render). "Manual" reveals a lightweight editor.
- **Manual editor = ▲/▼ arrows** (touch-safe) as the base, with optional pointer-only **drag handles** as an enhancement that degrades to arrows on narrow/touch widths. Numeric "run order" field is an equally good keyboard/touch editor if arrows feel clicky.
- **Persistence:** additive optional `questOrder` on the per-saga state object (`getSagaState`). Names absent from it fall back to canonical position, so partial orders and future-added quests just work. Stays v14 (additive) or bump to v15 for clarity.
- **Arc-aware:** for the ~8 multi-arc sagas, reorder *within* an arc and reorder the arcs themselves (`arcOrder` + per-arc `questOrder`), preserving the quest-giver/location arc headers rather than flattening them.
- **Group-by orthogonality (free win):** `groupBy` is saga-level, quest order is quest-level — a manual order survives regrouping with no extra work. Include a per-saga **reset-to-default**.
- **Effort:** S–M. Cleanly additive; drag re-wiring rides the existing `wireCard` re-attach pattern.

**Rejected / deferred alternatives:**
- **Drag-and-drop as the *primary* mechanism** — touch-hostile and higher effort; keep it as a pointer-only nicety over the arrow base, not the base itself.
- **Global name-keyed order overlay (cross-saga)** — a quest's rank would change meaning per container and fight arc grouping; confusing mental model. Rejected.
- **"Run Plan" stepper** (order as a route you walk through, ticking + auto-advancing to the next quest across sagas) — genuinely compelling but it's a *bigger, separate feature* that overlaps the prior doc's Next-Best-Action / Session-Planner ideas. **Deferred** to its own brainstorm; note the cross-link.

---

## Direction B — Autocomplete confirmation + session queue

**Recommended design (survivor): Confirm-each toggle + interactive detect-then-confirm toast with a tier picker + a foldable Session Run-Log tray (exportable).** This is the richest, most-wanted direction and it reinforces the earlier FTR fix.
- **Per-character setting (escape hatch):** extend prefs `{on, mode}` → `{on, mode, confirm}`, defaulting `confirm:true`. Slots into the existing `.autoc-group` radios. Users who liked the silent behavior pick "Auto-mark (no confirm)"; everyone else gets "Confirm each." *(from option #6 — trivial, safe additive field.)*
- **Confirm model:** split `_autoCompleteCheck` so a detection produces a **pending, non-fading interactive toast** ("Confirm / Dismiss") instead of committing; `_acApply` runs only on Confirm. Reuses the existing `.ac-toast` card + green/reaper variants; `_acSeen` already guarantees one detection per stay, so no toast spam across ticks. *(option #1 — respects the no-blocking-dialog harness rule.)*
- **Fold in the FTR-correctness layer:** the confirm isn't bare yes/no — it shows a small **tier picker (Normal / ☠ Reaper)** defaulting to the character's `mode`, so the player corrects difficulty *before* commit and FTR only seals when they actually reaped. This moves the exact decision the `wantReaper` gate was patched for to the moment of truth. Reaper chip pre-disabled for `NO_REAPER` quests; `reaperPopQueue` fed only on a Reaper confirm. *(option #4.)*
- **The session queue = promote `_acLog` into a rendered surface:** a collapsible **Run-Log tray** (twin of the existing `controlsPanel` fold via a `_runLogFolded` flag) accumulating every detection this session — quest · character · tier/reaper mark · timestamp · state chip (pending/confirmed/dismissed). *(option #3.)*
- **Export (nice-to-have):** persist the log across reload via the existing `_readAppMeta`/`_writeAppMeta` convention and add "Copy run log" / download — a shareable "tonight's run log" + FTR receipt, fully client-side. *(option #7.)*
- **Effort:** M overall (toggle is S; toast+picker S–M; tray M; export M). Single-file/offline clean, no schema migration — everything rides `_acLog`, the fold pattern, and the per-character prefs object.

**Rejected / deferred alternatives:**
- **Undo grace window** (silent apply + "Undo (12s)") — lowest friction, but the user explicitly asked to *confirm*, not to auto-apply-then-revoke. Keep as the fallback if per-detection confirm proves too clicky in play.
- **Pending-queue batch-approve** ("Approve all N") — great for raid nights, but the "nothing commits until you approve" latency can surprise. **Defer** as a power-user mode layered on the same `_acLog`.

---

## Direction C — Continued CPU efficiency (ranked; all cited)

The feature work above *adds* re-render triggers, which makes render-granularity wins more valuable, not less. Recommended order:

| # | Opportunity | Effort | Risk |
|---|-------------|--------|------|
| **C1** | **Expand/collapse toggles a CSS class, not a full re-render.** `wireCard`'s header handler calls `renderSagaList()`, but the `.saga-body` is *always* emitted — expand is purely presentational (`expanded` class). Replace with `card.classList.toggle("expanded")`. The single most-clicked interaction, near-free. | S | Low (byte-identical) |
| **C2** | **Don't reassign `_lfmIndex` on an idle poll.** The no-change branch of `refreshLfmIndex` still does `_lfmIndex = idx`, dropping the identity-keyed `lfmForTier` memo (the code even comments on it). Keep the existing reference → memo survives across idle polls. | S | Low |
| **C3** | **Compute `sagaStatus` once per saga per render.** Currently called 2–3× per saga (ftrOnly filter + card + rollup), each looping every quest. Thread one result through. | S | Low (byte-identical) |
| **C4** | **Stop double-rendering reaper/raid corners per poll.** `liveRefreshTick` unconditionally repaints them with stale data, then `refreshLfmIndex` repaints again on change. Drop the tick's redundant copies. | S | Low–M |
| **C5** | **Change-guard `renderChars`/`renderGuildCorner`/`renderServerPop`.** Unlike the LFM path, these rebuild `innerHTML` every 15s unguarded. Apply the same signature-guard already proven in-file. | M | Low |
| **C6** | **Precompute a per-saga `{normalizedName → questName}` map** so `_ftrNeeded` is O(1) instead of re-normalizing static names per reaper row per poll (next to `SAGAS_BY_NORM_QUEST`). | S–M | Low |
| **C7** | **Single-pass change signatures.** `sig`/`rsig`/`hsig` each do a full sort+map+join second pass per poll; accumulate a rolling hash inside the existing `forEach`. | M | Low |
| **C8** | **Structural single-card re-render** (the deferred Build-.1 finding). Re-render only the affected card(s) via `renderSagaCard` + `replaceWith` instead of tearing down the whole scroll on every checkbox/reward/skip. The completed-rollup toggle already demonstrates surgical updates in-file. Highest leverage; must re-render the affected *set* for `SHARED_QUESTS` cascades. | M | Medium |

**Already-optimal (negative signal):** background polling is correctly paused on `document.hidden` with a catch-up tick on `visibilitychange`; the DS&A indexes and the within-render `lfmForTier` memo from Build .1 are effective. No O(n²) remains in the lookup layer — the waste is render frequency/granularity, not the indexes.

---

## Batching recommendation (right-sized)
Three coherent builds rather than one:
1. **Perf foundation (fast, low-risk):** C1 + C2 + C3 + C4 (all S, byte-identical/low-risk) — pure performance, no behavior change; de-risks the feature builds that add re-render triggers. Optionally pull in C8 as its own unit.
2. **Confirm + session queue (Direction B)** — the meatiest, most user-facing, and it reinforces the FTR correctness fix.
3. **Custom quest ordering (Direction A).**

## Recommendation → next step (`/ce-brainstorm`)
- Take **Direction B (autocomplete confirm + session queue)** into `ce-brainstorm` first — it's the richest, most-requested, and pins down real design choices (confirm model, session boundary, export). 
- Ship the **C1–C4 perf quick wins** as a fast foundational build alongside or just before it (arguably `ce-plan` directly — they're well-specified and low-risk enough to skip brainstorm).
- Then **Direction A**.
- Defer the **"Run Plan" stepper** (Direction A) and **batch-approve** (Direction B) to their own future brainstorms; cross-link to `...-001`'s Next-Best-Action idea.
