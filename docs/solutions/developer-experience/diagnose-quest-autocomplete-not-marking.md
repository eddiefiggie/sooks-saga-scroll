---
title: Diagnosing "Quest Autocomplete didn't mark a quest" — the autocomplete → FTR → saga-reset pipeline
date: 2026-07-26
category: developer-experience
module: sooks-saga-scroll
problem_type: developer_experience
component: frontend_stimulus
severity: medium
applies_when:
  - "A user reports live Quest Autocomplete didn't mark, or marked the wrong container for, a quest"
  - "A report claims a behavior 'broke since the last build' and you need to confirm or rule out a regression"
  - "Investigating why an already-First-Time-Reaper quest won't re-take its completion seal after a saga reset"
tags: [quest-autocomplete, first-time-reaper, live-data, ddo-audit, debugging, regression, frontend]
---

# Diagnosing "Quest Autocomplete didn't mark a quest" — the autocomplete → FTR → saga-reset pipeline

> `component: frontend_stimulus` is only the schema's nearest frontend bucket — this app is vanilla single-file JS, no Stimulus. This is a knowledge/diagnostic reference, not a fix; the specific FTR-refire *bug* is [reaper-autocomplete-refires-ftr-after-seal-cleared](../logic-errors/reaper-autocomplete-refires-ftr-after-seal-cleared.md).

## Context

A report came in that live **Quest Autocomplete** stopped marking one quest ("A Raven at the Door") complete after the user reset its saga to re-run it — "maybe it broke in the last build." Triaging it meant reconstructing the whole autocomplete pipeline and its interaction with First-Time Reaper and saga resets. The report ultimately **did not reproduce** (it worked for another reset-and-rerun quest) and was treated as a one-off anomaly — so there is **no fix here**. The durable value is the pipeline map, the ordered diagnostic checklist, and the rule-outs that let a future report be triaged in minutes instead of an hour of code archaeology. All function/field names below are stable across builds; line numbers are not, so this cites names (verify against `index.html`, the live build).

## Guidance

### The pipeline (live tick → mark)

Quest Autocomplete is an opt-in per-character setting (`characterAutoComplete[char] = { on, mode }`, `mode` ∈ `normal | reaper`). On every live DDO Audit tick for the **active** character, `_autoCompleteCheck(char, locId)` runs this chain — any step returning early means **no mark and no toast** (silent):

1. **Enabled + active-char gate.** Bails unless `char === state.activeChar` and the per-character `on` is true.
2. **Detection — `getQuestAreaMap()`.** Resolves the character's live `location_id` to a quest name via the **reference cache** (DDO Audit's `areas`/`quests` lookup tables in `localStorage`). If the cache is stale or predates the content, the id isn't in the map → `qname` is `null` → early return. This is the most likely *silent* failure for one specific quest.
3. **Stay-guard — `_acSeen`.** A `char~normalizedQuest` key marks "already handled this visit," cleared only when the character leaves all quest instances (`qname` null). Prevents re-firing every tick during one stay.
4. **Name match — `normalizeQuestName()`.** Strips a leading `The`/`A`/`An` article, `(Epic)`/`(Legendary)` suffixes, and a `Return to ` prefix. Both the live name and the scroll's key normalize the same way ("A Raven at the Door" → "raven at the door"), so a mismatch here would make step 5 return nothing.
5. **Container selection — `_sagaForQuest(qname, lvl, lvl)`.** Picks **one** container by matching the **character level** to a tier band (`LFM_TIER_BANDS`: heroic `1–19`, epic `20–29`, legendary `30–99`). If the level fits no band, it falls back to `matches[0]` (index order). **Cross-tier versions are independent** — `SHARED_QUESTS` is keyed by `tier::questName`, so the Heroic and Legendary copies of a quest never cascade together, and resetting/marking one never touches the other.
6. **Apply — `_acApply(char, saga, q, mode)`.** Computes `alreadyDone = !!sagaDone[q]` (per-cycle green completion seal) and `alreadyReaper = quests[q] === "reaper"` (permanent FTR). Then `wantReaper = mode === "reaper" && !NO_REAPER.has(q) && !alreadyReaper`, and the guard `if (alreadyDone && !wantReaper) return null` no-ops the already-complete case. A non-null return drives `saveState()` + `renderSagaList()` + a toast.

### The two independent progressions (the crux)

A quest carries two separate flags, and every autocomplete question turns on which one you mean:

- **`sagaDone[q]`** — the green "Quest Completed" seal. **Per-cycle**: cleared when the saga reward is redeemed for a new run.
- **`quests[q] === "reaper"`** — First-Time Reaper. **Permanent, one-time**: survives resets forever.

So the "reset a previously-reaped quest, then re-run on reaper" case is *designed* to work: `alreadyReaper` is true → `wantReaper` degrades to `false` → the guard doesn't fire (because `alreadyDone` is false after the reset) → it re-applies just the green seal, no FTR re-pop. If that isn't happening, the failure is upstream (steps 2–5), **not** in the mark logic.

### Ordered diagnostic checklist for "it didn't mark quest X"

Read the saved state first — `localStorage.sooksSagaScroll` — then walk the pipeline:

1. **Is the green seal already set on the container it would resolve to?** If `sagaDone[q]` is true on that container, `_acApply` correctly no-ops. The real question becomes "why does that container still show complete?" → either the reset cleared a *different* container, or step 5 resolved the wrong one.
2. **Wrong-tier container (the prime suspect for multi-tier quests).** A quest that exists Heroic + Legendary resolves by *character* level. A high-level character running a lower-tier version, or a level that fits no band (falls to `matches[0]`), can mark/no-op the tier the user isn't running. Since cross-tier copies are independent, the container being run never gets touched.
3. **Detection gap.** If nothing at all happens (no toast on any container), suspect `getQuestAreaMap()` not resolving the `location_id` — a stale reference cache that doesn't know this quest's ids. The cache self-invalidates on a TTL and a content-epoch bump; a manual refresh (or epoch bump for new content) is the fix.
4. **Name normalization.** Confirm the live name and the scroll key normalize to the same string.
5. **Historical, old-import-only:** a load-time migration once re-derived `sagaDone[q]=true` from any truthy `quests[q]` on **every** load, re-sticking the seal onto a just-reset reaper quest. Fixed in **Build 06012026.1** by gating it to the one-time v6→v7 upgrade (`if (fromVersion < 7)`); inert for v7+ data. Only a suspect for "seal reappears after reset" on very old imported saves.

### Rule out a regression before chasing one

A "broke since the last build" report is a **hypothesis, not a fact** — verify it by diffing the actual functions across the two builds before investigating a regression. Here the autocomplete engine (`_acApply` / `_autoCompleteCheck`) was **byte-identical** between the two builds; the apparent diff was a pure line-number offset from unrelated CSS added above them. That single check redirected the whole investigation away from a phantom regression (and the report then didn't reproduce).

## Why This Matters

This subsystem has three conflatable concepts (FTR vs completion seal, character level vs quest tier, detection vs marking) and a deliberately silent failure mode (`_autoCompleteCheck` swallows every early return so live decoration never breaks the app). Without a map, each report costs an hour of re-derivation; with one, triage is a checklist. And the regression-rule-out habit prevents editing (or "fixing") code that never changed — the most expensive kind of wrong turn.

## When to Apply

- Any report that live autocomplete didn't mark, marked the wrong container, or a completion seal reappeared/won't re-apply.
- Any "it broke since the last build" claim — diff the specific functions first.
- Before changing anything in `_acApply` / `_autoCompleteCheck` / `_sagaForQuest`, to know what each step guards.

## Examples

Rule out a regression by diffing just the function bodies across two builds:

```sh
diff <(sed -n '/^function _acApply/,/^}/p' old-build.html) \
     <(sed -n '/^function _acApply/,/^}/p' new-build.html) && echo "unchanged — not this build"
```

Inspect saved state for a quest across every container (browser console, on the app with the user's data):

```js
const s = JSON.parse(localStorage.sooksSagaScroll || '{}'), c = s.activeChar;
console.table(Object.entries((s.progress || {})[c] || {}).flatMap(([id, ss]) =>
  [...new Set([...Object.keys(ss.sagaDone || {}), ...Object.keys(ss.quests || {})])]
    .filter(q => /raven at the door/i.test(q))
    .map(q => ({ container: id, greenSeal: !!(ss.sagaDone || {})[q], reaper: (ss.quests || {})[q], reward: ss.reward }))));
// container prefix h- vs l- = Heroic vs Legendary; compare against the character's level band.
```

## Related

- [reaper-autocomplete-refires-ftr-after-seal-cleared](../logic-errors/reaper-autocomplete-refires-ftr-after-seal-cleared.md) — the specific `_acApply` `wantReaper` fix (the FTR-refire bug) this pipeline notes build on.
- [derived-cache-reset-parity](../logic-errors/derived-cache-reset-parity.md) — a related reset/derived-state parity learning.
- `CONCEPTS.md` — "First-Time Reaper (FTR)" and "Reference cache" define the invariants this pipeline rests on.
