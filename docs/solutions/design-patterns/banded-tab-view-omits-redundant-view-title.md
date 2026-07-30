---
title: "A tab view whose content is already split into labeled bands should omit a redundant top-of-view title"
module: sooks-saga-scroll
date: 2026-07-30
problem_type: design_pattern
component: frontend_css
severity: low
resolution_type: code_fix
applies_when:
  - "A tabbed single-page view renders its content as a series of labeled section bands (tier headers, category headers)"
  - "The tab bar already names the view, and each band header already names its section"
  - "A view-level heading repeats the tab's own name above the bands, adding no information the bands don't already carry"
  - "A sibling tab in the same shell already renders headerless (content leads directly with its bands) and is the house pattern"
symptoms:
  - "Two stacked headings say almost the same thing — the tab name, then the first band's name right under it"
  - "One tab looks visually heavier at the top than its sibling tabs for no functional reason"
  - "A new banded tab copies the redundant-title shape and the inconsistency spreads"
tags:
  - tab-view
  - section-header
  - redundant-heading
  - visual-consistency
  - information-scent
related_components:
  - renderRaidsView
  - renderSagaList
  - renderPatronsView
---

# A tab view whose content is already split into labeled bands should omit a redundant top-of-view title

## Context

Sook's Saga Scroll is a four-tab single-page shell — **Saga · Raids · Patrons · Filters** — where the tab bar itself names the active view. Two of those tabs render their content as a stack of **labeled tier bands**:

- The **Saga** tab (`renderSagaList`, `index.html:15815`) leads straight into its tier sections. Each section's own header reads `Heroic` / `Epic` / `Legendary` — the tab goes tab bar → tier band, with **no** "Saga" title in between.
- The **Raids** tab (`renderRaidsView`, `index.html:18654`) renders the same shape: one section per band, each headed `${band} Raids` — `Heroic Raids`, `Epic Raids`, … (`index.html:18694`).

The Raids tab, however, opened with an extra view-level heading right under the tab bar:

```js
wrap.innerHTML = `
  <div class="tier-header raids-header"><h2>⚔ Raids</h2></div>
  <div class="patrons-charline">Keying &amp; completion for <strong>${escapeHtml(char)}</strong>
    <span class="patrons-sub">— keyed is derived from each raid's flagging quests</span></div>`;
```

So a user landing on the Raids tab saw, top to bottom: the highlighted **Raids** tab → a big **⚔ Raids** title → the keying charline → the **Heroic Raids** band header. The `⚔ Raids` title carried no information the tab and the band headers didn't already carry three times over. The Saga tab — the same author's earlier, more-iterated view — never had a "Saga" title; the Raids tab was simply built without noticing it had diverged from the house pattern.

## Guidance

When a tab's content is **already broken into labeled section bands**, lead with the bands. Do not add a view-level heading that repeats the tab's own name — the tab bar names the view and the band headers name the sections, so a third title in between is pure redundancy. Mirror whichever sibling tab established the headerless-banded pattern (here, the Saga tab).

The fix removed only the redundant title, keeping the keying charline (an information line — which character, and that "keyed" is derived — not a section header):

```js
// Build 07302026.1 — drop the redundant "⚔ Raids" section header. The per-band
// tier headers ("Heroic Raids", etc.) already label the sections, mirroring how the
// Saga tab goes straight from the tab bar into its tier bands (no "Saga" title).
wrap.innerHTML = `
  <div class="patrons-charline">Keying &amp; completion for <strong>${escapeHtml(char)}</strong>
    <span class="patrons-sub">— keyed is derived from each raid's flagging quests</span></div>`;
```

This is **not** a blanket "tab views never get a heading" rule. The distinction is whether the content is band-labeled:

- **Banded content** (Saga, Raids) → omit the view title; the bands are the labels.
- **Flat, unbanded content** legitimately keeps a single view title. The **Patrons** tab (`renderPatronsView`, `index.html:18702`) renders one flat favor list with no tier bands, so its `Patron Favor` heading (`index.html:18735`) is the *only* label its content has — that title stays. It is not a counter-example; it is the other side of the same rule.

Leaving the now-unused `.raids-header` CSS rule in place is fine — dropping the markup node is the whole fix; hunting the orphaned selector is unrelated churn.

## Why This Matters

A heading that repeats its own container's name is anti-scent: it spends the most visually prominent slot on the page restating what the user just clicked, pushing the actual content down and making one tab look heavier than its siblings for no reason. Worse, redundant chrome **replicates** — the next person adding a banded tab copies the nearest existing tab as a template, and if that template carries a vanity title, the whole shell drifts toward inconsistency one tab at a time. Enforcing "the bands are the labels" keeps every banded tab visually uniform and keeps the first band visible without scrolling.

## When to Apply

- Adding or reviewing any tab in a multi-tab single-page shell whose content renders as labeled section bands.
- Any time two headings stack with near-identical text (container name, then first section name) — treat it as a smell and ask which one is load-bearing.
- Deciding whether a tab-content view needs a title at all: it does **only** when its content is a flat list with no per-section headers of its own (the Patrons case).

## Examples

**Before** — Raids tab, three redundant name-echoes before any raid:

```
[ Saga | ‹Raids› | Patrons | Filters ]     ← tab bar names the view
             ⚔ Raids                        ← redundant view title (removed)
   Keying & completion for Sook …           ← info charline (kept)
          Heroic Raids                      ← band header already names the section
   • The Twilight Forge …
```

**After** — leads with content, matching the Saga tab:

```
[ Saga | ‹Raids› | Patrons | Filters ]
   Keying & completion for Sook …
          Heroic Raids
   • The Twilight Forge …
```

**Sibling reference (Saga tab, always been this way):**

```
[ ‹Saga› | Raids | Patrons | Filters ]
            Heroic                          ← straight into the band, no "Saga" title
   • Sinister Secret of Saltmarsh …
```

**The legitimate exception (Patrons tab — flat, unbanded):** keeps its `Patron Favor` title because that list has no per-section headers to label it.
