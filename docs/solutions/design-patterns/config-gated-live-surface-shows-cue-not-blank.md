---
title: "A config-gated live surface should show a 'not configured' cue, not a silent blank"
module: sooks-saga-scroll
date: 2026-07-29
problem_type: design_pattern
component: frontend_stimulus
severity: low
root_cause: config_error
resolution_type: code_fix
applies_when:
  - "A UI surface renders live/remote data only when a per-entity config value (server, account, feature key) is present"
  - "The live fetch early-returns on a falsy/unset config value, so the surface has nothing to render"
  - "The empty-config render path is identical to the genuine 'configured but currently no data' render path"
  - "A user cannot tell whether the feature is inert due to missing config vs. genuinely quiet right now"
  - "State carried over from a previously-active, fully-configured entity could leave stale data on screen"
symptoms:
  - "A feature panel/corner renders completely blank for some entities but works for others"
  - "\"Not configured\" is visually indistinguishable from \"no data at the moment\""
  - "Live data appears to work sometimes and silently do nothing other times, with no error"
  - "Stale live data from the previously-active entity lingers after switching to an unconfigured one"
tags:
  - empty-state
  - config-gating
  - live-data
  - discoverability
  - guard-clause
  - silent-blank
  - state-key
related_components:
  - renderRaidCorner
  - serverOf
  - refreshLfmIndex
  - state.characterServers
---

# A config-gated live surface should show a 'not configured' cue, not a silent blank

## Context

Sook's Saga Scroll shows a floating "raid LFM" corner that lists live raid groups pulled from the DDO Audit API for the active character's DDO server. That feed is gated on optional per-character config: the server is something the user sets by hand via `addCharacter(name, theme, server)` / `setCharacterServer(charName, server)` (`index.html:14211`, `index.html:14963`), stored in the `state.characterServers` map, and read back by `serverOf(charName)` — `return (state.characterServers && state.characterServers[charName]) || ""` (`index.html:14957`). It is NOT derived from whether the character is logged into the game; a live raid list depends only on the character having a *server* on record.

The fetch path bails on a missing server. `refreshLfmIndex(server)` opens with `if (!server) return;` (`index.html:15175`), so a character with no server assigned never populates `_raidLfms`. Before the fix, `renderRaidCorner()` guarded both the no-character and no-server cases with one silent line:

```js
if (!active || !server) { corner.innerHTML = ""; return; }
```

The result: a serverless character rendered an empty corner that looked exactly like a configured character with no raids currently posted. Two very different situations — "you haven't set this up" and "there's genuinely nothing right now" — collapsed into the same blank. Worse, because the empty path cleared `corner.innerHTML`, the failure mode was inconsistent depending on entry: switching from a server-set character to a serverless one could leave the prior character's raid rows on screen until the next render tick, so the surface could also carry stale data from a different entity.

## Guidance

Split the render guard so the config-missing state gets its own branch, and render a short inline cue there instead of blanking. The cue should name the missing config and the exact fix, and — critically — it must *replace* `corner.innerHTML` so no stale content from a previously-configured context survives.

Before (one guard, silent for both states):

```js
if (!active || !server) { corner.innerHTML = ""; return; }
```

After (`index.html:18075`–`18089`): keep the truly-empty case silent (no character selected is genuinely nothing to show), but give the unconfigured case a visible cue:

```js
if (!active) { corner.innerHTML = ""; return; }          // no character selected → nothing to show
if (!server) {
  corner.innerHTML =
    `<table class="guild-table" title="No DDO server is set for ${escapeHtml(active)} — live raid groups can't be fetched until one is chosen">` +
      `<thead><tr><th class="gm-fold"><span class="gm-caret">⚔</span> Raids</th></tr></thead>` +
      `<tbody><tr><td class="gm-empty">Set a server for ${escapeHtml(active)} to see live raids</td></tr></tbody>` +
    `</table>`;
  return;                                                 // before the live-list path
}
```

The cue reuses the corner's existing `guild-table` / `gm-empty` styling so it reads as part of the surface, not an error. Because it assigns `corner.innerHTML`, it wipes any raid rows left over from the previous active character in the same act.

Scope the cue deliberately. This same "hide on missing server" pattern lives in two sibling corners: `renderReaperCorner(band)` and `renderGuildCorner()` (`index.html:17927`), both of which still early-return `corner.innerHTML = ""` on `!active || !server`. The reaper surface renders three band cards (Low / Mid / High) from one renderer, so cueing there would repeat the identical "set a server" message three times for one character. The single raid-corner cue already tells the user this character has no server; adding it to the reaper trio would be noise, so the fix was intentionally scoped to the raid corner only. When N sibling surfaces share the same missing-config cause, cue once, in the most prominent one.

## Why This Matters

A silent blank is indistinguishable from "nothing to show." Users read it as "no raids right now" and never discover the feature needs a one-time server assignment — the capability is effectively invisible to anyone who hasn't already configured it. That is a discoverability failure. The stale-carryover variant is worse than invisible: showing the previous character's raid rows under the new character's name is actively misleading. Distinguishing "not configured" from "no data," and guaranteeing the cue replaces prior content, is both a discoverability win and an honesty win — the surface now tells the truth about its own state.

## When to Apply

Any per-entity live or remote surface that early-returns on missing config — a server, an API key, an account link, a region — and where an empty result is a legitimate, normal state that would otherwise be confused with "unconfigured." The tell is a single guard of the shape `if (!entity || !config) { blank(); return; }` protecting a fetch that itself no-ops on the missing config. Split it: keep the truly-empty case silent, but give the unconfigured case a named inline cue that points at the fix, and make sure that cue overwrites any content from a prior entity. If several sibling surfaces share the cause, cue once rather than repeating.

## Examples

The raid corner is the concrete case. Verified in-browser this session (Build 07292026.8):

- Active character "Sooktest" (server "Cormyr") → the real raid list renders, showing `▸ ⚔ Raids · 0` when no raids are posted — the honest empty-but-configured state.
- Switch to "Bee" (no server assigned) → the corner shows the inline cue "Set a server for Bee to see live raids," and NO stale raid rows carry over from Sooktest, because the cue path reassigns `corner.innerHTML`.
- Switch back to Sooktest → the real list returns. Zero console errors throughout.

Prevention note (probe the real state key). The first diagnostic pass raised a false "server selection is broken" alarm because the probe read state keys that don't exist — `state.charServers` / a `_server` field — and got empty / `"?"` results, which looked like a wiring bug. The production accessor reads `state.characterServers` (`serverOf`, `index.html:14957`); the config was fine all along. When verifying live-feature wiring, grep the accessor the production code actually uses and read *that exact key* — a wrong-key probe manufactures a false "broken" signal and sends the investigation chasing a bug that isn't there.

## Related

- [derived-per-character-status-reads-sagadone-not-quests](../logic-errors/derived-per-character-status-reads-sagadone-not-quests.md) — meta-lesson twin: a *derivation* read the wrong state field (`quests[q]` instead of `sagaDone`) and looked broken though the state was fine, exactly like the probe reading `charServers` instead of `state.characterServers`. Same failure family (wrong state key), different surface.
- [per-tick-live-render-needs-signature-guard](./per-tick-live-render-needs-signature-guard.md) — closest sibling: same `renderRaidCorner` / live-poll surface. That doc governs *when* to repaint (skip the rebuild when inputs are unchanged); this doc governs *what* to render when the surface is config-gated and unconfigured.
- [derived-cache-reset-parity](../logic-errors/derived-cache-reset-parity.md) — same live-LFM surface family (`refreshLfmIndex` / `_lfmIndex` feeding the floating corners); companion correctness gotcha for anything derived off the LFM fetch.
- [ddo-audit-exposes-real-party-status](../tooling-decisions/ddo-audit-exposes-real-party-status.md) — sibling floating-corner live surface (`renderGuildCorner`) with per-character config; both concern how a live DDO Audit corner should behave rather than silently degrade.
- [diagnose-quest-autocomplete-not-marking](../developer-experience/diagnose-quest-autocomplete-not-marking.md) — diagnosis-methodology twin: a "broke since last build" report that did not reproduce, mirroring the wrong-state-key probe raising a false "broken" signal.
