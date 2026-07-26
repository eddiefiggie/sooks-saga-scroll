---
artifact_contract: ce-unified-plan/v1
artifact_readiness: requirements-only
product_contract_source: ce-brainstorm
title: Real Group Status & Completed-Look Polish - Plan
date: 2026-07-26
module: sooks-saga-scroll
---

# Real Group Status & Completed-Look Polish - Plan

## Goal Capsule

- **Objective:** Rework the guild roster's group signal onto *reliable* live data and polish the completed-saga look. Ships as Build **07262026.4** (still schema v14; no reference-cache epoch change — `is_in_party`/`group_id` already ride the guild roster fetch).
- **Product authority:** the user (garage owner). Decisions below are user-directed this session.
- **Open blockers:** none. The key data question was resolved by a live probe: the DDO Audit guild endpoint's `online_characters` records carry `is_in_party`, `group_id`, and `is_recruiting` — so real party status (including private groups) is available with no new fetch.

## Product Contract

### Summary

Replace the LFM-derived "Pugging/blank" group signal with a real three-state Solo / Grouped / Guild Grouped read from the roster's own `is_in_party` + `group_id` fields, and apply four visual polish fixes to the banked-saga look and the LFM leader star.

### Problem Frame

Builds 07262026.1–.3 inferred grouping from the *advertised-LFM feed*, which can't see private/full groups — forcing a "Pugging / blank, never Solo" compromise and a `_groupIndex` cache. A live probe this session found the guild roster's own records already expose per-character party status, so the honest signal was there all along. Separately, the banked-saga treatment shipped in .3 needs colour/ribbon refinement and the LFM leader star is low-contrast on orange blocks.

### Requirements

- **R1 — Real three-state group status (row 4).** Derive each online guildmate's group state from the roster records: `is_in_party === false` → **"Solo"**; `is_in_party === true` with a `group_id` shared by ≥1 other online guildmate in the roster → **"Guild Grouped"**; `is_in_party === true` otherwise → **"Grouped"**. "Solo" is now shown (it is reliable). The row-4 slot stays reserved for uniform rows.
- **R2 — Guild-Grouped tooltip names the guildmates.** On hover of a "Guild Grouped" row, list the other online guildmates sharing that `group_id`. "Grouped" and "Solo" get a short explanatory tooltip.
- **R3 — Remove the LFM-derived group machinery.** Delete `_groupIndex`, its population in `refreshLfmIndex`, its reset sites, and the "Pugging" label — the group signal no longer depends on the LFM feed. The LFM feed keeps powering the reaper badges and cards; only the guild-roster group signal changes source.
- **R4 — Banked saga colour matches the quest zebra.** Change the banked-container background from the current darker gradient to the exact quest-row zebra dark tone (`rgba(101,78,40,0.26)`), flat.
- **R5 — Elegant diagonal corner ribbon.** Replace the hanging-pennant "✦ BANKED" ribbon with a diagonal banner that crosses the top-right corner (gold-trimmed emerald, majestic). The banked card clips the banner to the corner (e.g. `overflow:hidden` on the banked/complete card, verifying no wanted content is clipped).
- **R6 — White-outlined LFM leader star.** Give the gold leader ★ (`.lfm-star`) a white outline so it stays legible on the orange (mid) and red (high) LFM blocks.
- **R7 — Remove "Public Zone" from row 3.** Drop the `is_public` → "Public Zone" branch; keep the story/saga name (in a quest) and "Wilderness Zone" (`is_wilderness`); public or unknown areas leave row 3 blank. (`is_public` may stay in the areas cache; only the label is removed.)

### Key Decisions

- **Group status is real, three-state Solo / Grouped / Guild Grouped** (session-settled: user-directed), sourced from `is_in_party` + `group_id` on the guild roster's `online_characters` records (verified available by live probe this session). This supersedes the `Pugging`/blank compromise and its rationale.
- **"Guild Grouped" is defined by a within-roster `group_id` match** (session-settled: user-directed) — a party containing ≥1 other *online guildmate*, not merely being in any party.
- **`_groupIndex` and the LFM-derived group path are removed, not kept as a fallback** — the direct fields are strictly better (reliable, see private groups), so a fallback would only add carrying cost.

### Scope Boundaries

- No change to the reaper LFM badges/cards or the `_lfmIndex` beyond deleting the `_groupIndex` side-index.
- No reference-cache epoch bump — `is_in_party`/`group_id` are on the live guild fetch, not the cached areas/quests tables.
- No new "recruiting" state — `is_recruiting` exists but the chosen three states don't use it.

### Outstanding Questions

- Anonymous members: if an anonymous guildmate carries `is_in_party`/`group_id`, they still count toward a "Guild Grouped" match, but can't be named in the tooltip. Treat as best-effort (name omitted); not blocking.
