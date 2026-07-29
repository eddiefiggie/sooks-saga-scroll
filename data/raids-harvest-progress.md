# Raids Tab — harvest progress & method (Build 07292026.1)

The Raids-tab feature (U1–U3, U5, U6) is **built, verified in-browser, and committed**
with **2 seed raids** (Plane of Night, Tempest's Spine). This note captures the U4
harvest state so the remaining permanent raids can be added accurately.

## Authoritative roster (ddowiki Category:Raids, 41 pages)

**Permanent, to add** (band by quest level: 1–19 Heroic, 20–29 Epic, 30+ Legendary;
`Legendary <X>` titles are Legendary regardless):

- **Heroic:** Tempest's Spine ✅seeded · The Reaver's Fate · Zawabi's Revenge · The
  Titan Awakes · The Shroud · Hound of Xoriat · A Vision of Destruction · Ascension
  Chamber · Tower of Despair · The Chronoscope · The Lord of Blades · The Master
  Artificer · The Twilight Forge
- **Epic:** Plane of Night ✅seeded · Caught in the Web · The Fall of Truth
- **Legendary:** Legendary Tempest's Spine · Legendary Hound of Xoriat · Legendary
  Vision of Destruction · Legendary Master Artificer · Legendary Lord of Blades · The
  Chronoscope (Legendary) · Temple of the Deathwyrm · The Mark of Death · Defiler of
  the Just · The Codex and the Shroud · The Curse of Strahd · Old Baba's Hut · Project
  Nemesis · The Dryad and the Demigod · Den of Vipers · Relentless · Threats Old and
  New · Fire Over Morgrave · Riding the Storm Out

**Exclude (event/seasonal, R15):** Killing Time (Mabar), Fire on Thunder Peak, Hunt or
Be Hunted, Too Hot to Handle, Skeletons in the Closet (Night Revels). Verify each
against its wiki event category before final exclusion.

**Duplicate:** "The Vault of Night" (pageid 7601) is VoN 5 / the raid overview — the
raid instance is Plane of Night (already seeded). Do not add a second container.

## Harvest method (proven, reusable)

1. Roster: `api.php?action=query&list=categorymembers&cmtitle=Category:Raids&cmlimit=500&cmtype=page`.
2. Per-raid facts: `api.php?action=query&formatversion=2&prop=revisions&rvprop=content&rvslots=main&titles=<pipe-joined>`
   then parse the `{{Quest}}` infobox from wikitext. Field names: `level` (heroic
   level), `patron`, `adpack` (pack), `favor`, `zone` (region), `npc`/`bestower`.
   Flagging quests are the `[[links]]` in the `==Flagging==` section.
3. **Denoise flagging links** (they include spells/mechanics): keep only links whose
   page is in `Category:Quests by name` — `prop=categories&clcategories=Category:Quests%20by%20name%7CCategory:Raids`.
4. **In-page JS must use top-level `await`** (an `async` IIFE returns a Promise the REPL
   serializes as `{}`). Sanitize output of `= & ?` and URLs or the get-page guard blocks it.

## Verified batch-1 data (fields confirmed; use verbatim)

| Raid | Band | Lvl | Patron | Pack | Favor | Region | Flagging prereqs |
|---|---|---|---|---|---|---|---|
| The Reaver's Fate | Heroic | 14 | Agents of Argonnessen | Ruins of Gianthold | 6 | Gianthold | Madstone Crater, Prison of the Planes, A Cabal for One, Gianthold Tor |
| Zawabi's Revenge | Heroic | 12 | The Free Agents | Demon Sands | 6 | Zawabi's Refuge | Against the Demon Queen |
| The Titan Awakes | Heroic | 12 | The Free Agents | The Restless Isles | 6 | Restless Isles | none (no flagging) |
| Hound of Xoriat | Heroic | 18 | The Twelve | The Vale of Twilight | 9 | The Marketplace | none (no flagging) |
| A Vision of Destruction | Heroic | 18 | The Twelve | The Vale of Twilight | 9 | The Marketplace | none (no flagging) |
| Ascension Chamber | Heroic | 17 | The Silver Flame | The Necropolis, Part 4 | 7 | The Necropolis | The Litany of the Dead (pre-raid) |
| Tower of Despair | Heroic | 20 | The Yugoloth | The Devils of Shavarath | 8 | Amrath | none (no flagging) |
| The Chronoscope | Heroic | 6 | The Coin Lords | Devil Assault | 5 | The Marketplace | none (no flagging) |
| The Lord of Blades | Heroic | 20 | House Cannith | (confirm pack/favor) | – | House Cannith | none (no flagging) |
| The Master Artificer | Heroic | 19 | House Cannith | (confirm pack/favor) | – | House Cannith | none (no flagging) |
| The Twilight Forge | Heroic | 11 | (confirm) | The Restless Isles | – | Restless Isles | Hiding in Plain Sight, Slavers of the Shrieking Mines (+ Restless Isles quests — verify) |
| Caught in the Web | Epic | (epic) | (confirm) | Menace of the Underdark | – | Eveningstar | Complete the Menace of the Underdark story arc (all MotU quests) |
| The Fall of Truth | Epic | (epic) | (confirm) | (High Road/Gianthold) | – | Gianthold | Return to Madstone Crater, Return to Prison of the Planes, Return to … (verify) |

## Modeling decisions & gotchas (carry forward)

- **Band from quest level**, not the wiki raid category (the category API mis-suggested
  Chronoscope/Zawabi as Epic; their levels — 6 and 12 — are Heroic). `Legendary`-titled
  raids are Legendary. L20 boundary (Tower of Despair, Lord of Blades) modeled Heroic
  (Amrath/Cannith heroic raids); revisit if you prefer the app's 20→Epic band map.
- **`tier` = raidBand by default** so same-band prereqs cascade for free; set `tier` to a
  prereq's actual tier only when it lives in a Saga-tab container at a non-band tier
  (the Plane-of-Night → "Non-Saga Epic" seed pattern).
- **U6 cross-tier assertion is too strict** for the general harvest: a heroic raid whose
  prereq shares a NAME with an epic-tracked quest (e.g. Zawabi's "Against the Demon
  Queen") is a legitimately-different instance, not a harvest error. Relax the
  `runDataSelfCheck` cross-tier prereq check to a non-error note (or gate it to
  same-band) before bulk-adding, or it will emit false errors.
- **The Shroud** now flags via the **"The Thirteenth Eclipse"** story arc (reworked) —
  fetch that arc's quest list for its prereqs rather than the old Vale-flagging quests.
- Denoised flagging omits some entries (e.g. Reaver's Fate's "Prison of the Planes" was
  recovered only from the flagging prose) — always cross-check the prose, not just links.

## Remaining work

U4: add the ~31 permanent raids above (2 seeded). Then re-verify (`runDataSelfCheck`
0 errors incl. raid asserts; browser render + cascade), park (3-build retention), sync
`index.html`, update `README.md` resume prompt, and open the PR.
