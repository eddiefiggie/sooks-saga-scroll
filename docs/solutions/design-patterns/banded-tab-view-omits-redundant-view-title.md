---
title: "A banded tab view leads with its bands — drop redundant titles and per-view wrappers so sibling tabs align"
module: sooks-saga-scroll
date: 2026-07-30
last_updated: 2026-07-30
problem_type: design_pattern
component: frontend_css
severity: low
resolution_type: code_fix
applies_when:
  - "A tabbed single-page view renders its content as a series of labeled section bands (tier headers, category headers)"
  - "The tab bar already names the view, and each band header already names its section"
  - "A view-level heading repeats the tab's own name above the bands, adding no information the bands don't already carry"
  - "A sibling tab in the same shell already renders headerless (content leads directly with its bands) and is the house pattern"
  - "One tab's first section header sits at a different vertical offset than a sibling tab's, because the tabs use different container structure (a per-view wrapper vs. bands appended straight to the panel)"
symptoms:
  - "Two stacked headings say almost the same thing — the tab name, then the first band's name right under it"
  - "One tab looks visually heavier at the top than its sibling tabs for no functional reason"
  - "Switching between two tabs makes the first section header jump up or down because it's not at the same offset"
  - "A new banded tab copies the redundant-title or wrapper shape and the inconsistency spreads"
tags:
  - tab-view
  - section-header
  - redundant-heading
  - visual-consistency
  - cross-view-alignment
  - shared-structure
  - information-scent
related_components:
  - renderRaidsView
  - renderSagaList
  - renderPatronsView
---

# A banded tab view leads with its bands — drop redundant titles and per-view wrappers so sibling tabs align

## Context

Sook's Saga Scroll is a four-tab single-page shell — **Saga · Raids · Patrons · Filters** — where the tab bar itself names the active view. Two of those tabs render their content as a stack of **labeled tier bands**:

- The **Saga** tab (`renderSagaList`, `index.html:15815`) appends its tier-`section` bands straight to its panel (`#sagaList`), leading with the first band. Each section's own header reads `Heroic` / `Epic` / `Legendary` — tab bar → tier band, with **no** "Saga" title in between.
- The **Raids** tab (`renderRaidsView`, `index.html:18654`) renders the same band shape — one section per band, each headed `${band} Raids` (`Heroic Raids`, `Epic Raids`, …; `index.html:18691`).

The Raids tab was built with two pieces of chrome above its first band that the Saga tab never had, and they were removed across two consecutive builds:

1. **A redundant view title** (`⚔ Raids`) that just restated the active tab. Removed in Build 07302026.1 — the tab bar names the view and the band headers name the sections, so a third title in between carried no information.
2. **A keying/completion charline** ("Keying & completion for &lt;char&gt; — keyed is derived from each raid's flagging quests") wrapped in a `.patrons-wrap` view container. Build .1 *kept* this, reasoning it was an information line rather than a section header. But it still pushed the first `Heroic Raids` band lower than the Saga tab's first `Heroic` band, so the two tabs' headers no longer lined up when switching between them. Build 07302026.2 removed the charline **and its `.patrons-wrap` wrapper entirely**.

The alignment bug is the instructive part. The wrapper carried `.patrons-wrap { padding-top: 30px; }` (`index.html:4523`) with a comment saying it existed to "match the Saga tier header's ~40px gap below the tabs." It was an **approximation of another view's spacing** — and it was wrong by 10px, because the Saga tab's gap actually comes from the `.tier-section { margin: 40px 0 }` rule (`index.html:1168`) on its first band, not from any wrapper. Two tabs trying to look the same via independently-hand-tuned spacing will drift; they only stay aligned if they share the structure that produces the spacing.

## Guidance

**Lead a banded tab view with its bands. No redundant title, no per-view wrapper.** Append the section bands straight to the tab panel, exactly as the sibling tab that established the pattern does — then the *same* layout rule governs the first-header offset in every tab, and they align for free.

Build .2's `renderRaidsView` dropped the wrapper block entirely:

```js
// Build 07302026.2 — no view wrapper at all. The tier-section bands are appended
// straight to #raidsView, exactly like the Saga tab appends its bands to #sagaList,
// so the first band header ("Heroic Raids") lands at the same offset below the tab
// bar as the Saga tab's "Heroic" header — both governed by the shared 40px
// `.tier-section` top margin. This drops Build .1's keying/completion charline and
// its `.patrons-wrap` (which only approximated the gap with a 30px padding-top).
const raids = SAGAS.filter(s => s.isRaid);
// … RAID_BANDS.forEach → el.appendChild(sec) …
```

Two distinct rules are at work; keep them separate in your head:

- **Redundancy** — a view title that repeats the tab's own name is pure noise; drop it. This is *not* a blanket "tab views never get a heading" rule (see the Patrons exception below).
- **Cross-view alignment** — when two sibling views should present the same, do it by **sharing structure**, not by hand-tuning one view's spacing to approximate the other's. A wrapper that pads by 30px to "match" a 40px margin elsewhere is a latent drift bug; deleting it so both views hit the same `.tier-section` margin makes the alignment structural and self-maintaining.

**The legitimate exception — flat, unbanded content keeps its title.** The **Patrons** tab (`renderPatronsView`, `index.html:18699`) renders one flat favor list with no tier bands, so its `Patron Favor` heading (`index.html:18732`) is the *only* label its content has — that title stays, and this view legitimately keeps its own `.patrons-wrap`. It is not a counter-example; it is the other side of the rule. (This is why the `.patrons-wrap` CSS is kept even though the Raids tab no longer uses it.)

Leaving the now-unused `.raids-header` / `.raids-wrap` CSS rules in place is fine — dropping the markup nodes is the whole fix; hunting orphaned selectors is unrelated churn.

## Why This Matters

A heading that repeats its own container's name is anti-scent: it spends the most prominent slot on the page restating what the user just clicked, and pushes real content down. But the deeper, more transferable lesson is the alignment one: **visual consistency between sibling views is a property of shared structure, not of matching numbers.** The moment one view hand-codes a spacing value to look like another, the two are coupled by a constant that nothing enforces — a later change to the "source" view's margin silently breaks the copy, and the drift (here, 30 vs. 40px) is invisible until someone switches tabs and sees the header jump. Redundant chrome also **replicates**: the next person adding a banded tab copies the nearest existing tab as a template, so a vanity title or an approximating wrapper propagates the inconsistency one tab at a time. Deleting the wrapper so every banded tab bottoms out on the same `.tier-section` margin makes uniformity the default instead of a thing to re-tune.

## When to Apply

- Adding or reviewing any tab in a multi-tab single-page shell whose content renders as labeled section bands.
- Any time two headings stack with near-identical text (container name, then first section name) — treat it as a smell and ask which one is load-bearing.
- Any time you're tempted to add per-view padding/margin to make one view "line up with" another — stop and make them share the structure that produces the spacing instead. A magic constant that mirrors another view's value is a drift bug waiting to happen.
- Deciding whether a tab-content view needs a title at all: it does **only** when its content is a flat list with no per-section headers of its own (the Patrons case).

## Examples

**Before** (Build .0) — Raids tab, three redundant name-echoes plus a wrapper that misaligns the first band:

```
[ Saga | ‹Raids› | Patrons | Filters ]     ← tab bar names the view
             ⚔ Raids                        ← redundant view title  (removed in .1)
   Keying & completion for Sook …           ← charline in .patrons-wrap (removed in .2)
          Heroic Raids                      ← band header, pushed DOWN vs. the Saga tab
   • The Twilight Forge …
```

**After** (Build .2) — leads with the band; header sits at the Saga tab's offset:

```
[ Saga | ‹Raids› | Patrons | Filters ]
          Heroic Raids                      ← same vertical offset as Saga's "Heroic"
   • The Twilight Forge …
```

**Sibling reference (Saga tab, always been this way):**

```
[ ‹Saga› | Raids | Patrons | Filters ]
            Heroic                          ← straight into the band, no "Saga" title
   • Sinister Secret of Saltmarsh …
```

Switching Saga ⇄ Raids now leaves the first section header in place instead of nudging it up or down — the tell that the two views finally share structure rather than approximate each other.

**The legitimate exception (Patrons tab — flat, unbanded):** keeps its `Patron Favor` title and its own `.patrons-wrap` because that list has no per-section headers to label it.
