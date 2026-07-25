---
title: A content epoch invalidates a TTL + shape-guarded reference cache when upstream gains rows but keeps its schema
module: sooks-saga-scroll
date: 2026-07-24
problem_type: design_pattern
component: frontend_stimulus
severity: high
resolution_type: code_fix
applies_when:
  - "You cache an upstream reference table in localStorage with a time-based TTL (days) so live features resolve against it"
  - "A guard already refetches when the cached ROW SHAPE changes (a new field appears), but nothing refetches when only the row CONTENTS change (new rows added upstream)"
  - "New upstream content (a game update, a new catalog entry) must be reflected in the app quickly, not after the TTL lapses"
  - "Several features depend on the same cached table, so one stale cache breaks all of them at once"
tags:
  - caching
  - cache-invalidation
  - ttl
  - localStorage
  - reference-data
  - ddo-audit
  - staleness
---

# A content epoch invalidates a TTL + shape-guarded reference cache when upstream gains rows but keeps its schema

## Context

The guild-roster location line, live LFM badges, and quest Autocomplete are three
separate live features, but they all resolve the same way: a character's
`location_id` (an area id) or an LFM's `quest_id` is looked up against the **DDO
Audit reference tables** (`/areas`, `/quests`). Those tables are fetched once and
cached in `localStorage` under `sooksSagaScrollRef` (key `DDO_REF_KEY`, separate
from the app's own `state`) with a **7-day TTL** (`DDO_REF_TTL`). They are *not*
the hand-coded `QUEST_DETAILS` that render the saga cards — that distinction is
what made the bug confusing.

When DDO Update 81 "Terror of Demogorgon" went live (2026-07-22), every existing
user was holding a cache fetched **before** U81. All 13 U81 quests and their areas
were absent from the cached tables, so for the new content:

- the roster location line rendered blank — `questByArea.get(loc)` and
  `areasById.get(loc)` both missed (`renderGuildCorner`);
- U81 **LFM badges never appeared** — `refreshLfmIndex` drops any LFM whose
  `quest_id` it can't resolve to a name (`const qname = qmap.get(qid); if (!qname)
  return;`);
- quest **Autocomplete never fired** — `location_id → getQuestAreaMap()` returned
  nothing, so there was no quest name to hand to `_sagaForQuest`.

The upstream API had U81 the whole time (verified: pack "Terror of Demogorgon", 13
quests, all with valid `area_id`s), and the hand-coded container names matched the
API names exactly. Nothing in the app's own code or data was wrong — the only
defect was that a stale cache was being trusted.

## Guidance

When a reference cache is invalidated by **(a) a TTL** and **(b) a shape guard**,
add a third, explicit **content epoch**: a constant the code bumps whenever it
knows the upstream *contents* changed. Stamp it on every write; require it to match
on every read. A cache written under an older (or absent) epoch is treated as stale
and refetched once.

```js
// A monotonic integer bumped on each upstream content drop the build knows about.
//   1 = pre-U81 (implicit — caches with no epoch field)
//   2 = U81 "Terror of Demogorgon" (2026-07-22)
const DDO_REF_EPOCH = 2;

// Stamp on every write so a later load can tell what the cache predates.
function _writeRefCache(obj) {
  try { obj.epoch = DDO_REF_EPOCH; localStorage.setItem(DDO_REF_KEY, JSON.stringify(obj)); } catch (e) {}
}

// Require a match on read — an older/absent epoch falls through to a refetch.
if (cache.epoch === DDO_REF_EPOCH && cache.quests && cache.questsTs
    && (now - cache.questsTs) < DDO_REF_TTL && /* ...existing shape guard... */) {
  return _setQuestMaps(cache.quests /* ... */);
}
// ...else fetch fresh from the API, then _writeRefCache (which re-stamps the epoch).
```

The epoch is orthogonal to the TTL: the TTL bounds *staleness in general*
(week-old data refreshes on its own), while the epoch handles *a specific known
content change* immediately, for everyone, on their next load — without waiting for
the TTL and without forcing a global cache-key rename that would also nuke unrelated
cached data.

## Why This Matters

The pre-existing shape guard only refetched when a cached row **lacked a field** the
current build expected (e.g. a build that added `area`/`raid`/`pack` to each slim
row treated older caches as stale). That is a guard on **schema**, and U81 didn't
change the schema — a U81 quest row has the exact same fields as any other. Only the
**set of rows** changed. So both existing invalidation paths silently declined to
act: the TTL because the cache was days-fresh, the shape guard because the shape was
unchanged. The stale cache would have been trusted for up to the full 7 days.

The failure is easy to misdiagnose because the symptom surfaces in three unrelated-
looking features (roster, LFMs, autocomplete) and points at the *new content* —
tempting you to re-check the U81 data you just authored. The actual fault is one
layer down, in the shared cache all three consume. Naming that shared dependency
first collapses three bugs into one.

## When to Apply

- Any localStorage/IndexedDB/HTTP cache of a **reference/catalog table** with a
  multi-day TTL, where new upstream rows must show up promptly.
- Whenever you add a shape guard and catch yourself thinking "this refetches when
  the data changes" — it refetches when the *shape* changes. Content-only changes
  need the epoch.
- Prefer an epoch bump over renaming the storage key when the key also holds data
  you don't want to discard, or when you want the refetch to be a one-time
  correction rather than a permanent key migration.

## Examples

**Before** — two guards, both blind to content-only change:

```js
// TTL only; shape guard checks fields, not row membership.
if (cache.quests && cache.questsTs && (now - cache.questsTs) < DDO_REF_TTL
    && cache.quests[0].area !== undefined /* shape guard */) {
  return useCache(cache.quests);   // trusts a pre-U81 cache for up to 7 days
}
```

**After** — a content epoch forces a one-time refetch:

```js
if (cache.epoch === DDO_REF_EPOCH        // <-- content epoch
    && cache.quests && cache.questsTs && (now - cache.questsTs) < DDO_REF_TTL
    && cache.quests[0].area !== undefined) {
  return useCache(cache.quests);
}
// epoch mismatch (or absent) → fetch fresh, then _writeRefCache re-stamps epoch
```

**Verified end-to-end in-browser** (localhost + Claude-in-Chrome):

- Planted a stale **epoch-less** cache with a *fresh* timestamp and a valid shape —
  exactly what the old code would have trusted — then called `getQuestsMap()`. The
  guard rejected it and refetched the full 668-quest live table; U81 quests were
  present and `getQuestAreaMap().get(<U81 area_id>)` resolved to "Back to the Abyss"
  (the lookup the roster line and autocomplete both need).
- Planted a cache already stamped at epoch 2 → trusted without a needless refetch
  (a 1-row sentinel cache survived, proving no API hit). This is the negative
  control that keeps the epoch from turning into a refetch-every-load regression.

**Operational rule:** bump `DDO_REF_EPOCH` on every future DDO content drop that
adds quests or areas. The comment beside the constant is the changelog.

## Related

- [Cheap hot-path optimization over static data — memoize/key caches to object identity](memoize-static-data-key-caches-to-object-identity.md) — the other caching-discipline learning in this project; that one is about *in-memory* memo invalidation keyed to object identity, this one is about *persistent* reference-cache invalidation keyed to a content epoch.
- Live-data plumbing: `getQuestsMap` / `getAreasMap` / `getQuestAreaMap`, `refreshLfmIndex`, `renderGuildCorner`, `_acApply` — all consumers of the reference cache.
- Build 07242026.1 (schema v14, no schema change) — the build that introduced `DDO_REF_EPOCH`.
