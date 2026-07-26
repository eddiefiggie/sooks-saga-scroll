---
title: A derived module-level cache must be reset at every reset site of the cache it shadows
module: sooks-saga-scroll
date: 2026-07-26
problem_type: logic_error
component: frontend_live_data
severity: medium
symptoms:
  - "A newly added module-level cache (_groupIndex) kept stale cross-server data after a character switch because one of several reset sites was missed"
  - "The guild roster's Grouped/Solo line could briefly show a wrong-server group status after switching characters via the connect-switch banner, until the next LFM refetch rebuilt the index"
root_cause: logic_error
resolution_type: code_fix
tags:
  - live-data
  - cache-invalidation
  - reset-parity
  - lfm
  - gotcha
---

# A derived module-level cache must be reset at every reset site of the cache it shadows

## Problem

Build 07262026.1 added `_groupIndex` — a module-level cache of guild-member → group membership, built as a side effect of `refreshLfmIndex`'s existing `/lfms` fetch and read by `renderGuildCorner` to render the roster's Grouped/Solo line. Because it is derived from the same fetch as `_lfmIndex`, it must be cleared everywhere `_lfmIndex` is cleared. It was cleared at only two of the three reset sites; the third (`_connectSwitchTo`) kept a stale index.

## Symptoms

- After switching to a character on a **different server** via the connect-switch banner, the guild roster's line-3 status was computed against the *previous* server's group index for up to one tick.
- Because matching is by member **name**, the practical failure is a rare false "Grouped" on a cross-server name collision (or stale membership) — low-severity, but a correctness gap in a feature that is otherwise name-exact.

## What Didn't Work

- A single `Edit` with `replace_all: true` on the reset string `_lfmIndex = {}; _lfmFetchKey = ""; _lfmFetchTs = 0;` was assumed to cover every reset site. It updated the medallion-switch handler and the no-server path, but `_connectSwitchTo`'s reset line was **not** rewritten — so relying on the bulk replace, without independently enumerating and re-checking each reset site, silently left one path stale.

## Solution

Enumerate every place the shadowed cache is reset, then pair the derived cache's reset to each one. The missed site:

```js
// _connectSwitchTo — documented as replicating the medallion-click sequence 1:1
_nowPlaying = null; _locBannerDismissed = false;
// before: _lfmIndex = {}; _lfmFetchKey = ""; _lfmFetchTs = 0;
_lfmIndex = {}; _groupIndex = {}; _lfmFetchKey = ""; _lfmFetchTs = 0;   // after
state.activeChar = c;
```

Verification is a single grep — every `_lfmIndex = {}` line (except the declaration) must also clear `_groupIndex`:

```sh
grep -n '_lfmIndex = {}' sooks-saga-scroll-*.html | grep -v 'let _lfmIndex'
# each hit must contain "_groupIndex = {}"
```

## Why This Works

`_groupIndex` has the same lifetime and server-scope as `_lfmIndex` — both are rebuilt per server by `refreshLfmIndex` and are meaningless across a server change. `_connectSwitchTo` is explicitly documented as replicating the medallion-click switch "1:1"; the missed reset was therefore also a divergence from that stated contract, not just a cache bug. Clearing `_groupIndex` alongside `_lfmIndex` makes `renderGuildCorner` fall back to the safe default ("Solo") on the tick right after a switch, until the new server's LFM feed rebuilds both indexes.

## Prevention

- **When you add a cache derived from an existing one, treat its reset sites as a set, not a string.** Grep the shadowed cache's reset token, list every hit, and add the derived reset to each — don't trust one `replace_all` to have found them all (identical-looking lines can be missed, and near-identical ones won't match at all).
- **Prefer a single reset helper** when several fields must be cleared together. A `_resetLiveIndexes()` that clears `_lfmIndex`, `_groupIndex`, `_lfmFetchKey`, `_lfmFetchTs` in one place makes reset-parity structural instead of a discipline every call site must remember. (Deferred here to keep the build a minimal visual pass, but it is the durable fix.)
- **A "replicates X 1:1" comment is a testable claim.** When one path is documented as mirroring another, any state the mirrored path touches must be touched by both; a grep for each cleared symbol across both functions catches the drift.
