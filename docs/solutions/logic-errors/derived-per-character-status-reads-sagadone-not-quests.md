---
title: Derived per-character status must read sagaDone, not quests[q]
module: sooks-saga-scroll
date: 2026-07-29
problem_type: logic_error
component: frontend_state
severity: medium
symptoms:
  - "A newly added derived feature (raid keyed status) never updated when the user checked off a quest, even though completion state was clearly changing"
  - "Reading progress[char][id].quests[questName] returned false for quests the user had just marked complete via the left checkbox"
root_cause: logic_error
resolution_type: code_fix
tags:
  - completion-state
  - derived-state
  - sagadone
  - cascade
  - gotcha
---

## Problem

Any new per-character feature that derives from "has this quest been completed" must
read the `sagaDone` map, not `quests[q]`. The two are different fields with different
write paths, and the user-facing "Quest Completed" checkbox writes `sagaDone`. A derived
feature that reads `quests[q]` silently never reflects the user's checkbox clicks.

## Symptoms

- The Raids-tab keyed badge (derived: "keyed when all flagging quests complete") stayed
  at "Not keyed" no matter how many prereq quests the user checked in the raid card.
- A JS probe confirmed the completion WAS being recorded — but under
  `progress[char][id].sagaDone[q]`, while the derivation was reading
  `progress[char][id].quests[q]` (which stayed `false`).

## What Didn't Work

- Reading `getSagaState(char, id).quests[qName]` in the keyed helper. It looked correct
  (that's "the completion map") and even passed a first test — but only because that test
  wrote `quests[q]` directly. A real checkbox click never touches `quests[q]`, so keyed
  never flipped in actual use. The bug was invisible to a test that seeded the wrong field
  and only surfaced when the flow was driven through the real UI handler.

## Solution

Derive from `sagaDone`, the master "Quest Completed" seal:

```js
// WRONG — quests[q] is not what the completion checkbox writes
const q = getSagaState(char, raid.id).quests || {};
prereqs.forEach(p => { if (q[p]) done++; });

// RIGHT — sagaDone is the seal the left checkbox writes and Reaper/Skip auto-fill
const sd = getSagaState(char, raid.id).sagaDone || {};
prereqs.forEach(p => { if (sd[p]) done++; });
```

`computeCharacterFavor` (the patron-favor derivation) already did this correctly — it sums
Elite favor over quests whose `sagaDone[q]` is set. Any new derived per-character status
should follow that precedent.

## Why This Works

The app stores two independent per-quest fields in each container state
(`blankSagaState()` → `{ quests, sagaDone, skipDone, ... }`):

- **`sagaDone[q]`** — the master "Quest Completed" seal (per life). Written by the row's
  **left checkbox** (`toggleSagaDoneFor` → `setSagaDoneFor`) and auto-filled by the
  Reaper / Skip / Saga-Reward controls. It **cascades** across same-tier siblings via
  `SHARED_QUESTS[sharedKey(tier, q)]`, which is what makes shared-quest identity work
  across containers/tabs. This is the field that means "the player has completed this
  quest," so it's what every "did they do it" derivation must read.
- **`quests[q]`** — a `false | "normal" | "reaper" | "astral"` value tracking *how* a row
  is marked (Reaper seal, Astral/VIP skip, plain done). It is not the checkbox's target;
  `applyQuestState` writes it for the Reaper/Skip buttons. Reading it as a boolean
  "completed?" is wrong because the checkbox path leaves it `false`.

Because `sagaDone` cascades and `quests[q]` is written by a different control, a derived
feature reading `quests[q]` gets a value the primary completion UI never sets.

## Prevention

- **When adding a derived per-character "completion" feature, read `sagaDone`.** Grep for
  `computeCharacterFavor` as the reference implementation before writing a new one.
- **Verify derived UI through the real handler, not a seeded-state test.** The first test
  here passed only because it wrote `quests[q]` directly. Driving the actual checkbox
  (`input[type=checkbox][data-quest]` → its `change` handler) exposed the bug immediately.
  For this single-file app that means: localhost server + Claude-in-Chrome, click the real
  control, then read `sagaDone` vs `quests` from `state` — don't just assert on state you
  set yourself. (See [verify-single-file-html-without-node](../developer-experience/verify-single-file-html-without-node.md).)
- Related completion-state gotchas:
  [reaper-autocomplete-refires-ftr-after-seal-cleared](reaper-autocomplete-refires-ftr-after-seal-cleared.md)
  and [derived-cache-reset-parity](derived-cache-reset-parity.md).
