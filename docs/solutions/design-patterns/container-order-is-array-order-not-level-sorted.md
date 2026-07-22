---
title: Container display order within a tier group is SAGAS array order, not a render-time level sort
module: sooks-saga-scroll
date: 2026-07-22
problem_type: design_pattern
component: frontend_stimulus
severity: medium
applies_when:
  - "Adding a new saga container to the `SAGAS` array literal in index.html"
  - "The renderer groups containers by their `tier` field into a fixed `groupOrder`, then preserves array insertion order within each group"
  - "You expect containers to appear sorted by starting level, but there is no render-time sort by level"
  - "Deciding where in the `SAGAS` literal to insert a new container so it lands at the correct spot on the level ladder"
tags:
  - sagas
  - rendering
  - array-order
  - insertion-order
  - tier-grouping
  - data-convention
  - gotcha
---

# Container display order within a tier group is SAGAS array order, not a render-time level sort

## Context

Sook's Saga Scroll is a single-file app (`index.html`) whose entire dataset of
"containers" (sagas and non-saga groupings) lives in one big literal,
`const SAGAS = [ ... ]` beginning at `index.html:11530`, with five Non-Saga
Heroic containers `.push()`-ed onto it after the literal starting at
`index.html:12870`.

When you add a new container — which happens every time DDO ships an expansion or
update — the natural instinct is "it's Heroic, so add it to the Heroic block."
The trap: the Heroic block is a contiguous run at the *top* of the `SAGAS`
literal, so the path of least resistance is to drop the new entry at the group's
*start* (right after `const SAGAS = [`). That lands the new container at the very
top of the Heroic ladder on screen, regardless of its actual level — because the
render does not re-sort within a tier group. Where you splice the object into the
array literal *is* where it renders.

## Guidance

The rule: **insert each new container at its level-correct position inside its
tier's run of the `SAGAS` array.** Cross-group array position is irrelevant
(grouping keys off the object's `tier` field, so a Heroic entry can physically
sit anywhere in the array and still land in the Heroic section) — but
*intra-group* array position IS the on-screen display order.

Sort key for placement: **ascending by START level, ties broken by END level.**
For a single-level container (e.g. `levels: "11"`) treat start = end = that level.

Two small facts from the render path make this concrete. The tier group sequence
is a fixed, hardcoded list (`index.html:14776`):

```js
groupOrder = ["Heroic", "Non-Saga Heroic", "Non-Saga Epic", "Epic", "Legendary", "Non-Saga Legendary"];
```

and within the `groupBy === "tier"` branch the containers are bucketed in plain
array order with a `push`, no sort (`index.html:14778`):

```js
sagas.forEach(s => { if (!groups[s.tier]) groups[s.tier] = []; groups[s.tier].push(s); });
```

There is no `.sort()` anywhere in the tier path. (For contrast, the
`groupBy:"region"` and `groupBy:"pack"` branches DO call `groupOrder.sort()` at
`index.html:14784` and `index.html:14791` — but that alphabetizes the *section
headers*, not the containers within a section, and it only runs for those two
non-default modes. The default tier view sorts nothing.) The sections are then
emitted in `groupOrder` order and each section's cards are appended in bucket
(= array) order at `index.html:14794-14806`.

Because there is no level sort, the ascending order you see is *entirely* a
property of how the array literal is hand-maintained. Preserve it by hand on
every insert.

## Why This Matters

A mis-placed container produces **no error and no console warning** — it just
renders in the wrong spot on the level ladder and ships looking fine to anyone
not scanning levels top-to-bottom.

The data self-check does not catch it. `runDataSelfCheck()` at
`index.html:16916` validates *structure only*: unknown tiers, duplicate ids,
empty/duplicate quests, story-arc quests missing from the container's quest list,
and orphan ids in the `SAGA_STORIES` / `QUEST_DETAILS` / `QUEST_GIVERS` maps
(`index.html:16921-16933`). It returns `{ errors, info }` and never inspects the
`levels` field or the relative order of containers. So the green "✓ Data OK"
badge (`index.html:16953`) will happily display over a container sitting at the
wrong rung. Only a visual/level scan — or a runtime neighbor check — catches the
mistake.

## When to Apply

Any time a new saga or expansion container is added to `SAGAS` — which recurs
every DDO update. Before committing, confirm the new object sits between the two
existing containers whose start levels bracket its own, within its tier's run of
the array.

Pairs with [container-query-needs-ancestor-container](./container-query-needs-ancestor-container.md)
— its companion in the same tier-container subsystem in this file: that doc
governs how a container's header/rows *restack* by width (CSS container queries),
this one governs the *order* containers render in (JS array position). Also
touches the same render path as
[memoize-static-data-key-caches-to-object-identity](./memoize-static-data-key-caches-to-object-identity.md)
(`renderSagaList` / `SAGAS` / per-`tier` handling — same code region, different
concern). The ascending-order and corrected placement here were confirmed with
[the standard node --check + real-browser pass](../developer-experience/verify-single-file-html-without-node.md),
since — like the container-query gotcha — no syntax check or self-check flags a
wrong-order container.

## Examples

**DDO Update 81 — "Terror of Demogorgon"** added two containers: `h-demogorgon`
(Heroic, `levels: "11"`) and `l-demogorgon` (Legendary, `levels: "37"`).

- *First-draft (wrong) placement:* the implementation plan initially pointed both
  new containers at their group *starts* — the top of the Heroic run and the top
  of the Legendary run. That would have rendered the L11 container up at the L3
  slot and the L37 container up at the Legendary start (~L31) — both visibly out
  of order on the level ladder, with no error to flag it. The user caught it:
  "be sure this content is placed in a relative level-based order within the
  container sets."

- *Correct placement:* using ascending-by-start-level, `h-demogorgon` (L11) belongs
  between `h-mist` (`levels: "10-12"`, `index.html:11596`) and `h-myth-drannor`
  (`levels: "13"`, `index.html:11625`) — and it now sits there at
  `index.html:11612`. `l-demogorgon` (L37) is the highest Legendary entry, so it
  goes at the *bottom* of the Legendary run, immediately after `l-chill`
  (`levels: "36"`, `index.html:11915`) — and it now sits there at
  `index.html:11930`.

- *Verification:* an in-browser pass confirmed both tier groups render ascending
  by level, and a runtime neighbor check confirmed the ordering
  `h-mist → h-demogorgon → h-myth-drannor` and `l-chill → l-demogorgon → (end of
  Legendary)`.

For reference, the hand-maintained ascending Heroic sequence now reads (start
level, ties by end level): 3, 4-7, 5-6, 7-8, 8, 10-12, **11**, 13, 13-14, 15-16,
15-16, 15-19, 16-19, 18 — with `h-demogorgon`'s L11 slotted in after the L10-12
`h-mist`, exactly as the sort key dictates.
