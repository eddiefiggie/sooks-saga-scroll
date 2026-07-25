---
title: "Auditing saga/quest data against the DDO wiki: counts live on the saga page, names on the pack page"
module: sooks-saga-scroll
date: 2026-07-25
problem_type: developer_experience
component: development_workflow
severity: medium
applies_when:
  - "Verifying or updating the app's saga/quest container data against ddowiki.com (a content drop, or a suspected missing/misspelled quest)"
  - "You need the canonical quest NAMES for a saga, not just how many quests it contains"
  - "A direct WebFetch of a ddowiki page returns empty and you need another way to read it"
tags:
  - ddowiki
  - data-verification
  - saga
  - audit
  - browser-automation
  - parsing
  - gotcha
---

# Auditing saga/quest data against the DDO wiki

## Context

The app's saga containers (`quests[]`, `storyArcs`, `QUEST_DETAILS`, `QUEST_GIVERS`) are hand-sourced from ddowiki.com. Periodically the data needs an audit — after a content drop, or when a user reports a quest that looks missing or misspelled. Three non-obvious facts make a naive audit wrong.

## Guidance

### 1. The wiki splits the data across two pages

- A wiki **SAGA** page (e.g. `Magic_of_Myth_Drannor_(Heroic)`) gives the quest **COUNT** ("Consists of: N quests") and an inclusion rule in its Requirements section ("all pack quests except X", or "all of the … quests"). It usually does **not** list the quest names.
- The quest **NAMES** live on the adventure-**PACK** page (e.g. `Magic_of_Myth_Drannor`), grouped by story arc, and include raids / side quests the saga may exclude.
- So a **count-level** audit needs only the saga page; a **name-level** audit needs **both** (pack names, minus the saga's stated exclusions). Some saga pages *do* list names inline in Requirements — use them when present.

Each app container already stores its `wiki:` saga URL and its `quests[]` array — start the diff from those.

### 2. ddowiki is not WebFetch-able — use the browser

A direct `WebFetch` / `web_fetch` of a ddowiki page returns **empty** (proxy / anti-bot). Read pages with Claude-in-Chrome (`navigate` + `get_page_text`), and batch many pages per round-trip with `browser_batch` (sequential navigate→read pairs on one tab). localhost/ddowiki browsing permission is already granted for this project.

### 3. Parse the `quests[]` array quote-aware

Some quest names contain commas (e.g. `"Old Tomb, New Tenants"`). A naive `split(",")` of the `quests:[...]` array splits that one name into two entries — inflating the app count and manufacturing a phantom "extra quest" discrepancy. Extract quoted strings instead:

```js
(block.match(/"((?:[^"\\]|\\.)*)"/g) || []).map(s => s.slice(1, -1));
```

## Why This Matters

The count-vs-name split is the core trap: comparing the app count to the **pack** page's total over-counts (the pack includes quests the saga excludes), while names cannot be read from the saga page alone. The comma gotcha manufactures false positives. Getting either wrong wastes an audit — or, worse, "fixes" data that was already correct.

## When to Apply

- Any content drop (new expansion / pack) — re-audit the affected tiers.
- A user reports a missing or misspelled quest — verify against the pack page's exact spelling **before** editing; it may already be correct.
- Any script that reads the container `quests[]` arrays.

## Examples

The 2026-07-25 audit (Build 07252026.1) compared all 31 official saga containers (Heroic / Epic / Legendary) to their wiki saga pages by count + inclusion rule:

- **30 matched exactly.**
- **One real gap:** Heroic "The Pirates of the Thunder Sea" was missing the wiki-required quest **"Irestone Inlet"** (the app had 9; the wiki saga requires 10).
- A reported "Myth Drannor missing *Secrets of the Red Wizards*" was **already correct** (13/13; the plural spelling matched the wiki — no change made).
- A false "extra quest" in Epic Pirates was the comma-named **"Old Tomb, New Tenants"** mis-split by an early parser — the gotcha above.

Note: the **Non-Saga** containers (Non-Saga Epic / Legendary) are curated leveling/reaper groupings with no official wiki saga — the count audit does not apply to them; only name-level spot-checks against individual quest pages.

Cross-reference: the in-browser verification workflow this builds on — [verify-single-file-html-without-node](./verify-single-file-html-without-node.md).
