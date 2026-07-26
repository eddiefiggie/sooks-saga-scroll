---
title: DDO Audit exposes real per-character party status (is_in_party / group_id) — don't infer grouping from the LFM feed
module: sooks-saga-scroll
date: 2026-07-26
problem_type: tooling_decision
component: frontend_live_data
tags:
  - ddo-audit
  - live-data
  - api-shape
  - grouping
  - guild-roster
applies_when: "building or refining any feature that needs a live player's party/grouping status, or when tempted to build an inference layer around a perceived gap in an external API"
---

# DDO Audit exposes real per-character party status — don't infer grouping from the LFM feed

## Context

Builds 07262026.1 and .2 needed to show, on the online-guild roster, whether a guildmate was grouped. The only grouping source considered at the time was the **advertised-LFM feed** (`/lfms/{server}`) — the same feed powering the reaper badges. That feed only lists groups that are *posting* a "Looking For More", so a player in a full, private, or otherwise unadvertised group is invisible to it. That forced a hedged design: a best-effort `_groupIndex` (built by cross-referencing LFM leader/member names against the roster) and a "Pugging / blank" label that deliberately never said "Solo" (because absence of an advertised group is not evidence of solo).

A later live probe of the DDO Audit API (Build 07262026.4) found the real data had been available all along — on the roster feed the app already fetched.

## Guidance

The DDO Audit character records expose per-character party status directly. Both the whole-server feed **and the guild feed the roster already uses** carry these fields:

- `GET /v1/characters/{server}` — every online character record.
- `GET /v1/guilds/{server}/{guild}` → `data.online_characters` — the online-guildmate records the roster fetch already reads.

Each record includes:

- **`is_in_party`** (boolean) — grouped at all, including private/full groups the LFM feed can't see.
- **`group_id`** (number; `0` when solo) — *which* party. Two online members sharing a nonzero `group_id` are in the **same** party.
- **`is_recruiting`** (boolean) — the party is actively advertising (an open LFM).
- (plus `guild_name`, `is_anonymous`, `classes`, `location_id`, `total_level`, etc.)

Read grouping from these fields, not from the LFM feed. In this app, `_summarizeGuild` now picks `is_in_party` + `group_id` alongside the other member fields (see `sooks-saga-scroll-07262026-5.html`, `_summarizeGuild`), and `renderGuildCorner` builds a within-roster `group_id` → members tally so a member sharing a nonzero `group_id` with ≥1 other online guildmate reads "Guild Grouped" (and the tooltip names them), a lone party reads "Grouped", and `is_in_party === false` reads "Solo".

## Why This Matters

- **Correctness.** The per-character fields see every group; the LFM feed sees only advertised ones. "Solo" became an *honest* label only once it was backed by `is_in_party`, and "who are they grouped with" became answerable (shared `group_id`), including private groups.
- **Simplification.** The direct fields let Build 07262026.4 delete the entire LFM-derived `_groupIndex` machinery — its declaration, its capture inside `refreshLfmIndex`, and its reset sites — a net reduction in code and state. The LFM feed still powers the reaper badges/cards; only the guild-roster group signal changed source.
- **No new fetch.** The fields ride the guild roster fetch the roster already made, so this was a drop-in field pick, not a new network path or cache.

## When to Apply

- Any future feature needing a live player's party/grouping status — read `is_in_party` / `group_id` (and `is_recruiting` if you want an open-vs-closed distinction), don't reach for the LFM feed.
- The general lesson: **before building an inference layer around a perceived gap in an external API, probe the actual response shape.** A one-command dump of the real records (below) would have avoided the whole best-effort `_groupIndex` detour. The gap was assumed, not verified.

## Examples

Probe the real record shape (this is how the fields were found — WebFetch is proxy-blocked, but `curl` to `api.ddoaudit.com` works):

```sh
curl -s "https://api.ddoaudit.com/v1/characters/Cormyr" | python3 -c "
import sys,json,collections
d=json.load(sys.stdin); data=d.get('data',d)
rows=list(data.values()) if isinstance(data,dict) else data
rows=[r for r in rows if isinstance(r,dict)]
print('KEYS=', sorted(rows[0].keys()))
print('group-ish:', [(r['name'], r.get('group_id'), r.get('is_in_party')) for r in rows[:6]])
"
# KEYS include: group_id, is_in_party, is_recruiting, guild_name, is_anonymous, ...
# e.g. Ciarn group_id=144421459 is_in_party=True ; Gizmodon group_id=0 is_in_party=False
```

"Grouped together" via a within-roster `group_id` tally (directional):

```
// tally group_id -> [online members] over the roster, then per member:
//   !is_in_party                              -> "Solo"
//   group_id shared with >=1 OTHER member     -> "Guild Grouped" (name the others)
//   in a party, no other roster member in it  -> "Grouped"
```

Note: the guild feed returns `{data: {guild_name, server_name, character_count, online_characters}}` — the roster is under `online_characters` (a dict keyed by character id), not at `data` top level.
