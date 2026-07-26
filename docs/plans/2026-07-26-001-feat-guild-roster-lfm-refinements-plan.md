---
artifact_contract: ce-unified-plan/v1
artifact_readiness: requirements-only
product_contract_source: ce-brainstorm
title: Guild Roster & LFM Refinements - Plan
date: 2026-07-26
module: sooks-saga-scroll
---

# Guild Roster & LFM Refinements - Plan

## Goal Capsule

- **Objective:** Refine the live guild roster and the LFM indicators in Sook's Saga Scroll — a 4-row roster with story/zone context and an honest group signal, plus star-marked group leaders and clearer LFM labels. Ships as Build **07262026.2** (still schema v14; adds fields to the reference cache, which is versioned by its own epoch).
- **Product authority:** the user (garage owner). Decisions below are user-directed/approved this session.
- **Open blockers:** none — both data-feasibility questions were resolved by a live probe of the DDO Audit API (`is_public`/`is_wilderness` exist on `/areas`; quest→adventure mapping exists via `_questPackMap` and the app's own saga containers).

## Product Contract

### Summary

Restructure the online-guild roster into four rows (name+class/level, location, story/zone context, group status), fix the group signal to be honest about API visibility, and mark group leaders with a clear gold star across the roster tooltip and every LFM indicator. Show the quest name (not the area) on the saga-header LFM tag.

### Problem Frame

The roster and LFM displays overclaim and under-inform: "Solo" is asserted when the API genuinely can't see private groups; there's no story/zone context for what a guildmate is doing; group leaders aren't visually distinguished; and the saga-header LFM tag shows the adventure area instead of the more useful quest name. The live data supports fixing all of these.

### Requirements

- **R1 — Row 1 justification.** In each roster row, the character name is left-justified and the (abbreviated) class + level are **right-justified** on the same row. (Reverts Build 07262026.1's all-left row 1; class abbreviations from that build stay.)
- **R2 — Row 2 location (unchanged).** Keep the existing location line: `⚔ <quest name>` when in a quest instance, `📍 <area name>` when in a resolvable non-quest area.
- **R3 — Row 3 story/zone context (new).** A new third row per member: the **story/saga name** of the quest they're running; **"Wilderness Zone"** when the area is a wilderness; **"Public Zone"** when the area is public (non-wilderness); blank when none of these resolve. The row's slot is always reserved so rows stay uniform height.
- **R4 — Row 4 group status.** The group line moves to row 4: **"Pugging"** when the member is in an advertised LFM group; **blank** otherwise. Never render "Solo" — the LFM feed cannot see private/unadvertised groups, so absence is not evidence of solo. Slot reserved.
- **R5 — Guild header brackets.** Remove the `⟨ ⟩` brackets around the guild name in the roster header (`⟨Guild⟩ · N online` → `Guild · N online`).
- **R6 — Pugging tooltip.** The row-4 "Pugging" status shows, on hover, the **group size** and the **group leader**, with the leader marked by a **★**.
- **R7 — Leader star across LFM indicators.** Place a **★** immediately to the left of the group-leader name everywhere a leader is named: the right-side reaper/raid LFM cards, and the saga-header and quest-row LFM indicators. The star is gold and clearly star-shaped (★, U+2605).
- **R8 — Saga-header LFM tag names the quest.** The saga-header LFM tag's third line currently shows the adventure area/zone; show the **quest name** instead. (Requires carrying the quest name onto the per-group LFM record, which is keyed by normalized quest name today.)
- **R9 — Reference-cache enabler.** Add `is_public` and `is_wilderness` to the slim areas cache (currently `{id, name, region}` only) and bump `DDO_REF_EPOCH` (2 → 3) so any cache written under the old epoch refetches once and picks up the new fields. (Established pattern from Build 07242026.1.)

### Key Decisions

- **Group status is Pugging/blank, never Solo** (session-settled: user-directed). The only group signal available is the advertised LFM feed; a member in a full/private/unadvertised group is invisible to it, so "Solo" would be a false claim. "Pugging" is the honest label for an advertised (pick-up) group; absence renders blank.
- **"Story" = the saga/adventure the quest belongs to** (session-settled: user-approved), from the app's own saga-container data, falling back to the DDO Audit required adventure pack (`_questPackMap`) when the running quest isn't in a tracked saga.
- **Leader star = ★ (U+2605), gold** (session-settled: user-directed — "make it look clearly like a star").
- **Wilderness/Public classification comes from `is_public`/`is_wilderness` on the `/areas` record** (verified live this session), not inferred from the area name.

### Scope Boundaries

- **Not** building the per-character `group_id` probe against `/characters` — a true grouped/private/solo signal (including private groups) remains deferred. Pugging/blank is the ceiling for the LFM feed alone.
- **Not** changing the per-quest LFM badge that already shows the quest name on its own row.
- No schema (`SCHEMA_VERSION`/`STORAGE_KEY`) change — only the separate reference cache gains fields, guarded by its epoch.

### Outstanding Questions

- None blocking. If "story" should instead be the quest's synopsis blurb rather than the saga/pack name, that is a one-line swap flagged during the confirm step and not chosen.
