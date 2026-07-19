---
title: Cheap hot-path optimization over static data — index at load, and key memos/caches to object identity so invalidation is implicit
module: sooks-saga-scroll
date: 2026-07-19
problem_type: design_pattern
component: frontend_stimulus
severity: medium
resolution_type: code_fix
applies_when:
  - "A render/tick path re-runs linear scans (`.find` / `.filter` / nested normalize) over tables that are constant after load, multiplied across many rows"
  - "A value is recomputed many times per render from a source object that is reassigned (not mutated) whenever it changes — e.g. a poll rebuilds an index object each cycle"
  - "A change-detector re-serializes the current large value every tick just to compare it against the incoming one"
  - "You want a measurable CPU cut with byte-identical output and no new invalidation bugs"
tags:
  - performance
  - memoization
  - caching
  - hot-path
  - invalidation
  - data-structures
  - gotcha
---

# Cheap hot-path optimization over static data — index at load, and key memos/caches to object identity so invalidation is implicit

## Context

`renderSagaList()` tears down and rebuilds all ~53 saga cards / ~479 quest rows
on every click, autocomplete, and 15s poll tick. Per-row linear scans over the
static reference tables (`SAGAS.find`, `lookupQuestDetails`'s normalized
fallback, a double `normalizeQuestName` per quest) dominated CPU. The goal was a
real reduction with **byte-identical rendered output** and, critically, **no new
staleness bugs** — the usual failure mode when you add a cache to a hot path is
forgetting to invalidate it somewhere.

The durable lesson is three techniques that share one principle: **make cache
lifetime a property of an object's identity, not of a flag you have to remember
to reset.** Then invalidation is implicit and self-healing.

## Guidance

### 1. Index static tables once at load — no runtime invalidation

When a table is frozen after load (only an import/reset rebuilds *state*, never
the table itself), build its lookup Maps once and never invalidate them.

```js
// SAGAS / QUEST_DETAILS are constant after load.
const SAGA_BY_ID = new Map(SAGAS.map(s => [s.id, s]));   // was SAGAS.find(...) at ~8 hot sites

// Preserve the iteration order the old scans relied on → byte-identical results.
const QUEST_DETAILS_EXACT = new Map();   // questName -> details, first-occurrence
const QUEST_DETAILS_NORM  = new Map();   // normalizeQuestName(key) -> details, first-occurrence
for (const sid of Object.keys(QUEST_DETAILS)) {
  for (const key of Object.keys(QUEST_DETAILS[sid])) {
    if (!QUEST_DETAILS_EXACT.has(key)) QUEST_DETAILS_EXACT.set(key, QUEST_DETAILS[sid][key]);
    const nk = normalizeQuestName(key);
    if (!QUEST_DETAILS_NORM.has(nk)) QUEST_DETAILS_NORM.set(nk, QUEST_DETAILS[sid][key]);
  }
}
```

The equivalence argument is the load-bearing part: an index is only a safe drop-in
for a scan if it reproduces the scan's *first-match* semantics. Build the index in
the same iteration order the scan walked, and dedupe per key on first occurrence.

### 2. Generation-key a per-render memo to the source object's identity

When a value is recomputed many times per render from a source that is
**reassigned** (a fresh object) whenever it changes, memoize it and tie the memo's
lifetime to the source object's identity. It drops itself the moment the source is
reassigned — no manual `.clear()` call at the mutation site.

```js
let _lfmForTierMemo = new Map();
let _lfmForTierMemoGen = null;
function lfmForTier(questName, tier) {
  if (_lfmForTierMemoGen !== _lfmIndex) { _lfmForTierMemo.clear(); _lfmForTierMemoGen = _lfmIndex; }
  const key = questName + "|~|" + tier;          // separator that can't occur in the inputs
  const cached = _lfmForTierMemo.get(key);
  if (cached !== undefined) return cached;        // null is a valid cached result; undefined = miss
  const result = _lfmForTierCompute(questName, tier);
  _lfmForTierMemo.set(key, result);
  return result;
}
```

`lfmForTier` runs ~2× per quest per render (a header tint scan + the quest row).
Because the poll reassigns `_lfmIndex` to a **new object every cycle** (even when
unchanged), the memo naturally spans exactly one generation — long enough to serve
every call in a render, gone before stale data can leak. Two guardrails: distinguish
a cached `null` from a miss (`undefined`), and only share the returned reference if
callers treat it read-only.

### 3. Identity-key a signature cache so external resets self-heal

A change-detector that re-serializes the *current* large value every tick is doing
redundant work. Cache the signature, but pair it with the object it was computed
for — so it recomputes automatically if any *other* code path reset the source.

```js
// Before: sig(_lfmIndex) recomputed every tick.
if (sig(idx) !== sig(_lfmIndex)) { _lfmIndex = idx; renderSagaList(); }

// After: cache paired with the object; recompute only when that object changed.
if (_lfmSigFor !== _lfmIndex) { _lfmSig = sig(_lfmIndex); _lfmSigFor = _lfmIndex; }
const newSig = sig(idx);
if (newSig !== _lfmSig) { _lfmIndex = idx; _lfmSig = newSig; _lfmSigFor = idx; renderSagaList(); }
else                    { _lfmIndex = idx; _lfmSig = newSig; _lfmSigFor = idx; }
```

`_lfmIndex` is also reset to `{}` elsewhere (e.g. the active character has no
server). A bare cached `_lfmSig` would go stale against that reset; pairing it with
`_lfmSigFor` (the object the signature belongs to) makes the guard recompute the
moment the identity no longer matches — correct **without** coupling the cache to
every reset site.

## Why This Matters

The recurring bug when you add a cache to a hot path is a missed invalidation: some
distant code mutates or resets the source and your cache keeps serving the old
answer. All three techniques above sidestep that by making **object identity** the
cache key:

- A **static table** never changes identity, so its index never needs invalidation.
- A **per-generation source** changes identity on every update, so an identity-keyed
  memo self-expires exactly once per generation.
- An **externally-resettable source** changes identity on reset, so an identity-keyed
  signature cache self-heals without knowing about the reset sites.

You get the CPU win of caching without taking on the coupling and staleness risk
that usually comes with it — and, for the static-table case, the output stays
byte-identical, which is what makes it a safe drop-in.

## When to Apply

- The scan/recompute is on a genuinely hot path (per-row in a full re-render, or
  per-tick in a poll) — not a cold path where the linear version is clearer.
- For indexing (technique 1): the table is provably constant after load. If it can
  mutate at runtime, you need real invalidation and this shortcut does not apply.
- For the memo (technique 2): the source is **reassigned** on change, not mutated in
  place. Identity-keying only works if a change produces a new object.
- For the signature cache (technique 3): the current value can be reset by code paths
  you don't want to couple to.

## Examples

- **Byte-identical refactor discipline.** All three techniques preserve behavior, so
  they were verified by confirming the rendered DOM and mark/cascade behavior were
  unchanged (a clean in-browser render on a crafted test character), not by testing
  new behavior. Treat "prove it's identical" as the acceptance bar for this class of
  change, not "prove the new feature works."
- **Live file:** the build these landed in is `sooks-saga-scroll-07192026-1.html`
  (functions `lfmForTier` / `_lfmForTierCompute`, `lookupQuestDetails`,
  `_sagaForQuest`, `refreshLfmIndex`, and the index block after `SHARED_QUESTS`).
- **Related:** `docs/solutions/design-patterns/gpu-cheap-continuous-css-animations.md`
  is the CSS/paint-side companion to this JS/data-side pass — both cut the same
  app's render cost from different layers. Verification approach:
  `docs/solutions/developer-experience/verify-single-file-html-without-node.md`.
