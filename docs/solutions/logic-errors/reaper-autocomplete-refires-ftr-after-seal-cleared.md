---
title: Reaper autocomplete re-fires First-Time Reaper after the completion seal is cleared
module: sooks-saga-scroll
date: 2026-07-19
problem_type: logic_error
component: frontend_stimulus
severity: medium
symptoms:
  - "Re-running a quest already completed on Reaper (its FTR banked) re-pops the ☠ First-Time-Reaper animation and toasts \"First-Time Reaper sealed automatically\" a second time"
  - "With reaper-mode Quest Autocomplete on, entering an already-reapered quest whose completion seal was cleared (saga redeemed → new cycle) celebrates FTR again instead of just marking the quest complete"
root_cause: logic_error
resolution_type: code_fix
tags:
  - reaper
  - first-time-reaper
  - autocomplete
  - quest-completion
  - state-cycle
  - gotcha
---

# Reaper autocomplete re-fires First-Time Reaper after the completion seal is cleared

## Problem

First-Time Reaper (FTR) is a **permanent, one-time** claim, but ordinary quest
completion is a **per-cycle** state that gets cleared when a saga is redeemed and
re-run. A mark path that treats "should I seal FTR?" as `mode === "reaper"`
alone — without checking whether FTR is *already* claimed — will re-celebrate FTR
every time the quest is re-marked in a later cycle. In `sooks-saga-scroll` this
surfaced through reaper-mode Quest Autocomplete: re-entering an already-reapered
quest whose completion seal had been cleared re-fired the ☠ pop and the
"First-Time Reaper sealed" toast, even though the credit was banked long ago.

## Symptoms

- Re-running an already-reapered quest (new saga cycle) pops the ☠ FTR animation
  and toasts "First-Time Reaper sealed automatically" again.
- The two progressions a quest carries are independent and stored separately:
  `quests[q] === "reaper"` (FTR — permanent) vs. `sagaDone[q]` (completion seal —
  cleared when the saga reward is redeemed for a new cycle). The bug lives at the
  point where these two get conflated.

## What Didn't Work

- The original `_acApply` guard `if (alreadyDone && (!wantReaper || alreadyReaper)) return null;`
  correctly no-ops the *fully-done* case (already reaper **and** already sealed),
  so the bug is invisible there. It only appears in the asymmetric state —
  `alreadyReaper && !alreadyDone` — which the guard lets through, and then
  `wantReaper` (computed as `mode === "reaper" && !NO_REAPER.has(q)`) is still
  `true`, so the function re-adds to `reaperPopQueue` and returns `reaper: true`.
  Patching only the guard would not fix it; the defect is that `wantReaper`
  ignores whether FTR is already claimed.

## Solution

Gate the "seal FTR" intent on FTR **not** already being claimed, at the point
where `wantReaper` is computed — so the whole function degrades to a plain
completion for an already-reapered quest. In `_acApply`:

```js
// Before
const wantReaper = mode === "reaper" && !NO_REAPER.has(qName);
const alreadyDone   = !!(ss.sagaDone && ss.sagaDone[qName]);
const alreadyReaper = (ss.quests || {})[qName] === "reaper";
if (alreadyDone && (!wantReaper || alreadyReaper)) return null;

// After
const alreadyDone   = !!(ss.sagaDone && ss.sagaDone[qName]);
const alreadyReaper = (ss.quests || {})[qName] === "reaper";
const wantReaper = mode === "reaper" && !NO_REAPER.has(qName) && !alreadyReaper;
if (alreadyDone && !wantReaper) return null;
```

Reorder so `alreadyReaper` is known before `wantReaper`, then AND it in.
Because `wantReaper` is now `false` for an already-reapered quest, the FTR-seal
write and `reaperPopQueue.add(qName)` are skipped and the returned `reaper` is
`false`, so the caller (`_autoCompleteCheck`) shows the normal "Quest Completed"
toast instead of the ☠ one. The green completion seal still cascades and
auto-banks exactly as a normal completion. The guard simplifies to
`if (alreadyDone && !wantReaper) return null;` — behaviour-equivalent to the old
guard given the new `wantReaper`, since `!alreadyReaper` now folds into it.

## Why This Works

The single source of "is this a first-time reaper?" becomes `!alreadyReaper`, and
it is applied once, upstream of every downstream effect (the FTR seal write, the
pop queue, the return `reaper` flag, and therefore the toast). So the three
states resolve correctly:

| State | `wantReaper` | Result |
|-------|--------------|--------|
| Not done, not reaper, mode reaper | `true` | Seal FTR + green seal, ☠ pop, "FTR sealed" toast |
| Reaper claimed, seal **cleared** (re-run) | `false` | Green seal only, **no** ☠ pop, "Quest Completed" toast |
| Reaper claimed **and** sealed (fully done) | `false` | No-op (`return null`) |

Verified in-browser by driving `_acApply` through all three: the middle case
returns `reaper:false` with `reaperPopQueue` untouched and `quests[q]` still
`"reaper"`; the first still pops; the third no-ops.

## Prevention

- **Treat FTR as write-once.** Any code path that *seals* First-Time Reaper —
  the ☠ REAPER button (`applyQuestState("reaper")`), autocomplete (`_acApply`),
  or any future mark path — must condition the seal (and its celebration) on
  `quests[q] !== "reaper"`, never on `mode === "reaper"` alone. FTR is permanent;
  the completion seal (`sagaDone[q]`) is per-cycle and will legitimately toggle
  back to unset. Never infer "first time" from the completion seal's state.
- **Test the asymmetric state, not just the extremes.** A quest has two
  independent flags, so there are four combinations. Bugs hide in the mixed ones
  (`reaper && !sagaDone`), not in "both set" or "both clear". When adding a mark
  path, exercise all four.
- Cross-reference the FTR invariant in `CONCEPTS.md` ("First-Time Reaper") — it
  now states FTR persists across completion-seal clears.

## Related Issues

- Plan: `docs/plans/2026-07-19-001-feat-autocomplete-fix-cpu-and-bar-visuals-plan.md` (unit A).
- Verification workflow: `docs/solutions/developer-experience/verify-single-file-html-without-node.md`
  (the `node --check` + in-browser `_acApply` drive used to confirm this fix).
