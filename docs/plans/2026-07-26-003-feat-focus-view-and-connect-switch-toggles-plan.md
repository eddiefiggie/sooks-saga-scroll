---
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
product_contract_source: ce-plan-bootstrap
execution: code
title: "feat: Focus View + Connect-Switch toggles in Filters & Alerts - Plan"
date: 2026-07-26
module: sooks-saga-scroll
---

# feat: Focus View + Connect-Switch toggles in Filters & Alerts - Plan

## Goal Capsule

- **Objective:** Add two user-facing toggles to the Filters & Alerts panel, tucked in like the existing items: (1) a visible on/off for the existing **connect-switch prompt** (which currently has a setting but no UI), and (2) a new **Focus View** toggle that master-switches the location-emphasis feature and makes location-matched sagas **pulse**. Ships as Build **07262026.5** (still schema v14; settings persist in app-meta, not the versioned state).
- **Product authority:** the user (garage owner). Scope was settled in this session's brainstorm; decisions are labeled below.
- **Verification reality:** this project has **no automated test harness** — verification is the built-in data self-check plus an in-browser drive (localhost + Claude-in-Chrome), per the project's established method. "Test scenarios" below are in-browser verification scenarios, not unit tests.
- **Open blockers:** none. One assumption to confirm: **Focus View default ON** (see KTD2).

---

## Product Contract

### Summary

Surface the deferred connect-switch on/off as a clean checkbox, and add a Focus View toggle that gates the whole location feature (banner + highlight + a new pulse) so a player can see, at a glance, which tracked sagas match where their character currently is.

### Problem Frame

The connect-switch prompt (offers to switch the scroll to a character you have online when your tracked one is offline) ships with a working setting but **no UI** — the code says *"until a settings checkbox lands"*. Separately, the location feature (a "📍 you're in X" banner + a `.saga-now-playing` highlight on matching sagas) is **always on** with no off switch and only a static highlight. Both gaps live in the same place: the Filters & Alerts panel, which already hosts Audio Alerts and Quest Autocomplete toggles.

### Requirements

- **R1 — Connect-Switch toggle.** A checkbox in Filters & Alerts reflects and controls the existing connect-switch setting: checked ⇔ `_connectSwitchEnabled()`; toggling calls `_setConnectSwitchEnabled(checked)`. Default **ON** (unchanged). Carries a `title=` explanation of what the prompt does.
- **R2 — Focus View toggle.** A new checkbox in Filters & Alerts, persisted in app-meta (new key, e.g. `focusView`), default **ON**. Carries a `title=` explanation of what Focus View does.
- **R3 — Focus View is the master switch for the location feature.** When **ON**: matched sagas pulse **and** keep the existing highlight, and the "📍 in X" banner shows. When **OFF**: no pulse, no `.saga-now-playing` highlight, and no location banner.
- **R4 — Location pulse.** When Focus View is ON, the saga containers matching the active character's current area (the existing `sagasMatchingArea` / `_nowPlaying.sagaIds` set) **pulse**. The pulse is GPU-friendly (animate `opacity`/`transform`, not `box-shadow`), applies only to matched sagas, and is stilled under `prefers-reduced-motion` — consistent with the app's CPU/animation passes.
- **R5 — Live + persistent.** Both toggles show their current state on load (from app-meta) and take effect immediately on change (the location emphasis re-renders without a reload; the connect-switch re-evaluates on its next tick / immediate re-render as the existing setter already does).
- **R6 — Clean placement + explanation.** Both toggles use the existing `.control-group` / `.audio-check` markup and styling so they read as siblings of Audio Alerts and Quest Autocomplete, each with its own one-line `title=` explanation.

### Key Technical Decisions

- **KTD1 — Focus View gates the ENTIRE location feature (banner + highlight + pulse), not just the pulse.** (session-settled: user-directed — chosen over "Focus View only adds the pulse".) Rationale: one clean "show me where I am" switch, and it finally gives the always-on location feature an off switch.
- **KTD2 — Focus View defaults ON.** (session-settled: user-directed — chosen over default-OFF/opt-in.) Rationale: the location banner/highlight is always-on today; defaulting the master switch OFF would silently remove behavior users currently have. Default ON preserves it and adds the pulse.
- **KTD3 — Connect-Switch toggle reuses the existing setting; no new state.** (session-settled: user-directed.) `connectSwitchPrompt` and `_setConnectSwitchEnabled` already exist; this is UI only.
- **KTD4 — Pulse is CPU-conscious.** (session-settled: user-directed — "CPU-conscious pulse".) Animate a GPU-composited property, matched sagas only (usually 0-few), honor `prefers-reduced-motion`. No continuous page-wide animation.
- **KTD5 — Settings live in app-meta, like `connectSwitchPrompt`.** No `SCHEMA_VERSION`/`STORAGE_KEY` change; app-meta round-trips via the existing whole-state export.

### Scope Boundaries

- **Not** changing what the connect-switch prompt does or how it detects candidates — only exposing its on/off.
- **Not** changing the `sagasMatchingArea` matching logic — the pulse reuses whatever `_nowPlaying.sagaIds` already resolves.
- **Not** adding new location data sources or per-region tuning.

#### Deferred to Follow-Up Work

- A visible master control for the connect-switch was previously deferred; this plan delivers it. No new deferrals.

---

## Implementation Units

### U1. Add the two toggles to the Filters & Alerts markup

- **Goal:** Render a Connect-Switch checkbox and a Focus View checkbox in the Filters & Alerts panel, styled as siblings of the Audio Alerts / Quest Autocomplete groups.
- **Requirements:** R1, R2, R6.
- **Dependencies:** none.
- **Files:** `sooks-saga-scroll-07262026-5.html` (the new build; the Filters panel markup around the `control-group audio-group` / `autoc-group` blocks).
- **Approach:** Add a new `.control-group` (e.g. a "View & Alerts" group) — or extend an existing group — containing two `.audio-check` `<label>`s, each wrapping a `<input type="checkbox">` with a stable id (e.g. `focusViewToggle`, `connectSwitchToggle`) and a `title=` explaining the behavior. Match the existing 2×N grid so it doesn't disturb the panel layout.
- **Patterns to follow:** the `audio-group` / `autoc-group` blocks (`<div class="control-group X-group"><label>Title</label><div class="audio-checks"><label class="audio-check" title="…"><input type="checkbox" id="…"> Label</label></div></div>`).
- **Test scenarios (in-browser):** the two checkboxes appear in Filters & Alerts, aligned with the other groups; hovering each shows its explanatory tooltip; no layout regression to the existing controls at desktop and narrow widths.
- **Verification:** self-check clean, 0 console errors, the panel renders both toggles cleanly.

### U2. Wire the Connect-Switch toggle to the existing setting

- **Goal:** The Connect-Switch checkbox reflects and controls `connectSwitchPrompt`.
- **Requirements:** R1, R5.
- **Dependencies:** U1.
- **Files:** `sooks-saga-scroll-07262026-5.html` (control wiring alongside the other Filters checkbox handlers).
- **Approach:** On render/load, set the checkbox `checked` from `_connectSwitchEnabled()`. On `change`, call `_setConnectSwitchEnabled(checkbox.checked)` (which already persists to app-meta and re-renders the guild corner / connect-switch banner). No new state.
- **Patterns to follow:** the existing Audio Alerts / `acEnabled` checkbox wiring in the controls binding.
- **Test scenarios (in-browser):** toggling OFF then reloading keeps it OFF (persisted); with a synthetic offline-selected-char + online-alt state, the connect-switch banner appears when ON and does not when OFF.
- **Verification:** the setting round-trips across reload and gates the banner.

### U3. Add the Focus View setting + wire its toggle

- **Goal:** A persisted `focusView` app-meta setting (default ON) with a read helper, and its checkbox wired.
- **Requirements:** R2, R5.
- **Dependencies:** U1.
- **Files:** `sooks-saga-scroll-07262026-5.html` (an app-meta read/write helper pair mirroring `_connectSwitchEnabled` / `_setConnectSwitchEnabled`, plus the checkbox handler).
- **Approach:** Add `_focusViewEnabled()` (reads `_readAppMeta().focusView`, `undefined → true` default ON) and `_setFocusViewEnabled(on)` (writes app-meta, then re-renders the location emphasis). Reflect state on the checkbox at load; on `change`, call the setter.
- **Patterns to follow:** `_connectSwitchEnabled()` / `_setConnectSwitchEnabled()` and the `connectSwitchPrompt` app-meta pattern.
- **Test scenarios (in-browser):** default state is checked on a fresh load; toggling persists across reload.
- **Verification:** `_focusViewEnabled()` returns true by default and reflects the toggle; setting round-trips.

### U4. Gate the location feature on Focus View

- **Goal:** When Focus View is OFF, suppress the location banner and the `.saga-now-playing` highlight; when ON, keep them.
- **Requirements:** R3, R5.
- **Dependencies:** U3.
- **Files:** `sooks-saga-scroll-07262026-5.html` (the `setNowPlaying` path / the now-playing trigger where `setNowPlaying(charName, area)` is called, and the banner + `.saga-now-playing` render).
- **Approach:** Gate the now-playing computation and render on `_focusViewEnabled()`: when OFF, do not set `_nowPlaying` (or clear it) so no banner and no highlight paint; when the toggle flips, re-run the location resolution / re-render so it appears or disappears immediately. Keep the existing `sagasMatchingArea` logic intact.
- **Execution note:** verify the OFF path clears an already-painted banner/highlight (toggle OFF while a location is active), not just the not-yet-painted case.
- **Patterns to follow:** the existing `setNowPlaying` / `clearNowPlaying` / `_nowPlaying` render and the `renderConnectSwitch` enable-gate.
- **Test scenarios (in-browser):** with a synthetic active area that matches a saga — ON shows the banner + highlight; toggling OFF removes both immediately; toggling back ON restores them.
- **Verification:** banner + highlight track the toggle live.

### U5. Add the location pulse

- **Goal:** Matched sagas pulse when Focus View is ON.
- **Requirements:** R4.
- **Dependencies:** U4.
- **Files:** `sooks-saga-scroll-07262026-5.html` (CSS: a pulse keyframe + a class on matched sagas; render applies it only when Focus View is ON).
- **Approach:** Add a GPU-friendly pulse (animate `opacity`/`transform` on the matched saga container or an overlay pseudo-element — not `box-shadow`), applied to the `_nowPlaying.sagaIds` sagas (reuse or extend `.saga-now-playing`). Gate the pulse on `_focusViewEnabled()` (already true whenever `_nowPlaying` is set, per U4). Wrap the animation in `@media (prefers-reduced-motion: no-preference)` so it stills under reduced motion (fall back to the static highlight).
- **Execution note:** mirror the 07182026.9 CPU pass — animate a composited property, never a continuous page-wide `background-position`/`box-shadow` loop; only the (usually few) matched sagas animate.
- **Patterns to follow:** the connect-switch banner's `::after` opacity-pulse (GPU-friendly glow) and the `prefers-reduced-motion` gating used on the progress-bar sheen.
- **Test scenarios (in-browser):** matched sagas visibly pulse when Focus View is ON; non-matched sagas do not; under `prefers-reduced-motion` the pulse stills to the static highlight; 0 console errors; no visible CPU thrash (only matched sagas animate).
- **Verification:** pulse appears on matched sagas only, respects reduced-motion, and disappears when Focus View is OFF.

---

## Verification Contract

- Data self-check reports **0 structural errors** (64 containers / 541 quests / 100% detailed) — unchanged by this UI feature.
- **0 console errors** on load and while toggling.
- In-browser drive (localhost + Claude-in-Chrome): both toggles render cleanly in Filters & Alerts with tooltips; Connect-Switch toggle gates the banner and persists; Focus View toggle gates banner + highlight + pulse, persists, and defaults ON; pulse is matched-sagas-only and stills under reduced motion.
- `node --check` on the extracted script passes (JS changes present).

## Definition of Done

- U1-U5 implemented in a copy-first new build (`sooks-saga-scroll-07262026-5.html`), verified in-browser per the Verification Contract.
- Both settings persist in app-meta and round-trip via export; no schema/`STORAGE_KEY` change.
- Parked per the garage convention (retention, README park line + ladder + Resume prompt, index.html sync), reviewed, committed, and pushed (publishes via Pages).

## Open Questions

- None. Focus View default resolved to **ON** (KTD2, session-settled).
