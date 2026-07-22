---
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
execution: code
product_contract_source: ce-plan-bootstrap
title: "feat: Add Update 81 — Terror of Demogorgon (2 sagas, 12 quests)"
created: 2026-07-22
plan_type: feat
schema_impact: none (stays v14)
sources:
  - data/update-81-terror-of-demogorgon-research.md
---

# feat: Add Update 81 — Terror of Demogorgon (Heroic + Legendary sagas, 12 quests)

## Summary

Add DDO Update 81 ("Terror of Demogorgon") content to Sook's Saga Scroll as **two
new saga containers** — `h-demogorgon` (Heroic, L11) and `l-demogorgon`
(Legendary, L37) — each holding the same **12 quests** grouped into three story
arcs (Part One ×5, Part Two ×5, Side ×2). All content is **directly sourced from
the DDO wiki** (`data/update-81-terror-of-demogorgon-research.md`); nothing is
inferred. This is a **data-only** change: new entries in `SAGAS`,
`SAGA_STORIES`, `QUEST_GIVERS`, and `QUEST_DETAILS`, plus the data self-check
count bump. **No schema change — stays v14, `STORAGE_KEY` untouched.**

Per the two scoping decisions taken during planning: **build now with the
preview-data gaps flagged** (4 placeholder synopses + 2 placeholder locations
become explicit `TODO` markers with source URLs, closed by a re-source unit), and
**populate full `QUEST_DETAILS` depth for all 12 quests** (verbatim story +
complete monster arrays + levels + no-traps), matching how existing packs
(Saltmarsh, Vecna, etc.) are stored.

---

## Problem Frame

The tracker currently holds 58 containers / 517 quest slots across six tiers.
Update 81 shipped 2026-07-22 with a new Underdark expansion whose quests are not
yet represented. The tracker's value is completeness against live DDO content,
so U81 must be added following the established data contract exactly, so that
multi-character progress, LFM, story arcs, quest givers, and per-quest detail all
work with zero schema disruption.

**Non-goal:** No new UI, layout, or mechanic. Terror of Demogorgon is a
conventional dual-tier saga pack and must slot into the existing container/arc/
detail rendering with no code-path changes beyond data.

---

## Sources & Research

- **Primary dossier:** `data/update-81-terror-of-demogorgon-research.md` — the
  full sourced capture (expansion overview, both sagas, all 12 quests with
  givers/areas/synopses/monsters), each datum carrying its `ddowiki.com` source
  URL. Captured 2026-07-22 via Claude-in-Chrome MCP (direct WebFetch of
  `ddowiki.com` is proxy-blocked; `www.ddo.com` is fully blocked — Chrome MCP is
  the only working path, as recorded in project memory).
- **⚠️ Data maturity:** every wiki page still carried the **Lamannia
  preview-server banner** ("speculation until live") at capture time, and 4
  synopses + 2 locations are wiki placeholders. See U6 (re-source pass) and Risks.

---

## Key Technical Decisions

- **KTD1 — Two containers, not one.** Model as `h-demogorgon` (tier `Heroic`) +
  `l-demogorgon` (tier `Legendary`), mirroring the existing dual-tier packs
  (`h-vecna`/`l-vecna`, `h-isle-dread`/`l-isle-dread`). The wiki confirms two
  distinct sagas with different bestowers and level ranges. *(session-settled:
  user-directed — chosen over a single merged container.)*
- **KTD2 — `QUEST_DETAILS` defined once, on the Heroic key.** Verified against
  the codebase: legendary containers carry their own `SAGAS`, `SAGA_STORIES`, and
  `QUEST_GIVERS` entries but **no** `QUEST_DETAILS` block — the legendary side
  resolves story/monsters through `lookupQuestDetails()` cross-tier fallback +
  `normalizeQuestName()`. U81 quest names are identical across both tiers (no
  `(Legendary)` suffix), so a single `QUEST_DETAILS["h-demogorgon"]` block of 12
  quests serves both containers. Do **not** duplicate details under
  `l-demogorgon`.
- **KTD3 — Per-quest levels `{heroic:11, legendary:37}`.** Uniform across all 12
  (wiki: Heroic 11 / Epic 37 "considered Legendary"). Container `levels` = `"11"`
  (Heroic) / `"37"` (Legendary).
- **KTD4 — Story arcs carry `giver: null`.** Each quest has its own distinct
  bestower (all in Gravenhollow), so there is no single arc-giver; per-quest
  givers live in `QUEST_GIVERS`. This matches how "Side Quests" arcs are already
  modeled. The **saga** bestower (Haruhak Deepgift / Eramura Orefinder) is the
  container `npc`.
- **KTD5 — Build now, flag gaps.** Integrate all sourced content immediately; the
  4 placeholder synopses and 2 placeholder locations become explicit inline
  `TODO(u81-resource): <url>` markers, closed by U6 once the live wiki updates.
  *(session-settled: user-directed — chosen over waiting for a live re-source.)*
- **KTD6 — `source` / `wikiUrl` on every record.** Every `SAGA_STORIES.source`
  and `QUEST_DETAILS.wikiUrl` points at the exact `ddowiki.com` page, preserving
  auditability and the "never fabricate" rule.

---

## Data Contract Reference (target shapes, from live `index.html`)

```
SAGAS[]         : { id, tier, name, levels, npc, npcLocation, region, pack,
                    patron, wiki, quests[], storyArcs[{name,giver,giverLocation,quests[]}] }
SAGA_STORIES[id]: { synopsis, source }
QUEST_GIVERS[id]: { "<questName>": "<giverNPC>" }
QUEST_DETAILS[id]["<questName>"]:
                  { levels:{heroic,legendary}, story, monsters:[{name,url}],
                    trapsNone:true, wikiUrl }
```

Container ids: `h-demogorgon`, `l-demogorgon`. Group placement: `h-demogorgon`
into the **Heroic** group, level-ordered **between `h-mist` (L10-12) and
`h-myth-drannor` (L13)**; `l-demogorgon` into the **Legendary** group,
level-ordered **at the bottom, after `l-chill` (L36)**. Group order (Heroic →
Non-Saga Heroic → Non-Saga Epic → Epic → Legendary → Non-Saga Legendary) is
unchanged; intra-group order is array insertion order (no render-time level
sort — `index.html:14601`), so the array position of each new object is what
places it on the level ladder.

**Canonical quest set (both containers, 12):**
Part One — Demon in the Rough · Tower of Vengeance · A Blood Pact · Hideous to
Behold · The Price of an Egg. Part Two — For Want of a Heart · Flocked Together ·
The Fetid Wedding · Stealing from Sorcere · Back to the Abyss. Side — A Long Way
to Mushrooms · If It Wasn't For Bad Luck.

---

## Implementation Units

### U1. Add `h-demogorgon` + `l-demogorgon` to `SAGAS`

**Goal:** Two new container objects with quest lists and three story arcs each.
**Requirements:** KTD1, KTD3, KTD4.
**Dependencies:** none.
**Files:** `index.html` (the `SAGAS` array literal).
**Level-ordered insertion points (CRITICAL — intra-group order is array order, no
render-time level sort; verified `index.html:14601`):**
- `h-demogorgon` (L11) goes in the **Heroic** block **between `h-mist` (L10-12)
  and `h-myth-drannor` (L13)** — i.e. after `h-mist`'s closing brace, before the
  `h-myth-drannor` object. NOT at the top of the Heroic block. The Heroic ladder
  is ascending by start level (ties by end level); L11 sits after L10-12, before
  L13.
- `l-demogorgon` (L37) goes at the **bottom of the Legendary** block **after
  `l-chill` (L36)** and before the Non-Saga Epic block (`ns-e-phiarlan-carnival`)
  begins. L37 is the new group max, so it renders last in Legendary.
- The 5 Non-Saga Heroic containers are `.push()`-ed after the literal and live in
  a separate tier group; they do not affect Heroic-group ordering.
**Approach:**
- `h-demogorgon`: `tier:"Heroic"`, `name:"Terror of Demogorgon"`, `levels:"11"`,
  `npc:"Haruhak Deepgift"`, `npcLocation:"Gravenhollow"`, `region:"The Underdark"`,
  `pack:"Terror of Demogorgon"`, `patron:"Battlehammer Expedition"`,
  `wiki:"https://ddowiki.com/page/Terror_of_Demogorgon_(Heroic)"`.
- `l-demogorgon`: same but `tier:"Legendary"`, `levels:"37"`,
  `npc:"Eramura Orefinder"`,
  `wiki:"https://ddowiki.com/page/Terror_of_Demogorgon_(Legendary)"`.
- `quests[]` (both) = the 12 canonical names above, in arc order.
- `storyArcs[]` (both, identical):
  `{name:"Terror of Demogorgon Part One", giver:null, quests:[5 P1]}`,
  `{name:"Terror of Demogorgon Part Two", giver:null, quests:[5 P2]}`,
  `{name:"Side Quests", giver:null, quests:[A Long Way to Mushrooms, If It Wasn't For Bad Luck]}`.
**Patterns to follow:** existing `h-vecna`/`l-vecna` and `h-isle-dread`/
`l-isle-dread` container pairs.
**Test scenarios:** (visual/behavioral — no unit-test harness in this single-file
app; see Verification)
- Both containers render in the correct tier groups (Heroic / Legendary).
- Each shows 12 quests across 3 labeled arcs; Side arc shows 2 quests.
- Container header shows bestower + Gravenhollow + patron correctly.
- Per-character progress checkboxes persist across reload (localStorage v14).
**Verification:** container count 58 → 60; each new container fully expandable.

### U2. Add `SAGA_STORIES["h-demogorgon"]` + `["l-demogorgon"]`

**Goal:** Saga-level synopsis + source for each container.
**Requirements:** KTD6.
**Dependencies:** U1.
**Files:** `index.html` (`SAGA_STORIES` map ~L5323+).
**Approach:** Both entries get a synopsis drawn from the sourced expansion
overview (Underdark / Gravenhollow / driving the Demon Lords back to the Abyss via
Vizeran's Dark Heart ritual), `source:"https://ddowiki.com/page/Terror_of_Demogorgon"`.
Distinguish Heroic (bestowed by Haruhak Deepgift, L11) vs Legendary (Eramura
Orefinder, L37) in the synopsis text, matching the h-/l- phrasing pattern used by
Vecna's two entries.
**Patterns to follow:** `h-vecna` / `l-vecna` `SAGA_STORIES` entries.
**Test scenarios:** synopsis renders on both container headers; source link
resolves.

### U3. Add `QUEST_GIVERS["h-demogorgon"]` + `["l-demogorgon"]`

**Goal:** Per-quest bestower NPC for all 12, in each container.
**Requirements:** KTD4.
**Dependencies:** U1.
**Files:** `index.html` (`QUEST_GIVERS` map ~L11291+).
**Approach:** Both containers get the same 12 mappings (wiki lists identical
bestowers per tier): Demon in the Rough→Dreline the Trader; Tower of Vengeance→
Grin Ousttyl; A Blood Pact→Graz'zt the Dark Prince; Hideous to Behold→Vizril
Geemor; The Price of an Egg→Jurdor the Provisioner; For Want of a Heart→Brumo
Zivthin; Flocked Together→Regmur the Learned; The Fetid Wedding→Mobopho; Stealing
from Sorcere→Altnil T'orgh; Back to the Abyss→Kleve; A Long Way to Mushrooms→
Dirnun the Healer; If It Wasn't For Bad Luck→Ervo the Curious.
**Test scenarios:** each quest row shows its correct giver in both tiers.

### U4. Add `QUEST_DETAILS["h-demogorgon"]` — full depth, 12 quests

**Goal:** Verbatim story, complete monster arrays, levels, and no-traps for all
12 quests (Heroic key only; Legendary resolves via fallback per KTD2).
**Requirements:** KTD2, KTD3, KTD6.
**Dependencies:** U1.
**Files:** `index.html` (`QUEST_DETAILS` map ~L5399+).
**Approach:** One block keyed `"h-demogorgon"` with 12 quest sub-objects. Each:
`levels:{heroic:11, legendary:37}`, `story:"<verbatim wiki synopsis>"`,
`monsters:[{name,url}]` from the sourced monster lists (URL =
`https://ddowiki.com/page/<Monster_Name>`), `trapsNone:true` (all 12 are "no
traps"), `wikiUrl:"https://ddowiki.com/page/<Quest_Page>"`.
- For the **4 placeholder synopses** (Hideous to Behold, For Want of a Heart, The
  Fetid Wedding, Stealing from Sorcere) use a concise sourced-content summary and
  append `TODO(u81-resource): <quest wiki url>` — do **not** invent story text.
- Do **not** add an `l-demogorgon` `QUEST_DETAILS` block (KTD2).
**Execution note:** After adding, exercise `lookupQuestDetails('l-demogorgon', <name>)`
in-browser to confirm the legendary container resolves all 12 quests' details via
fallback (target: 12-for-12), the same way Legendary Vecna does.
**Patterns to follow:** existing `QUEST_DETAILS["h-saltmarsh"]` structure.
**Test scenarios:**
- All 12 Heroic quest detail panels show story + monster links + "no traps".
- All 12 **Legendary** quest detail panels show the same detail via fallback (no
  blank panels) — this is the KTD2 integration proof.
- Monster wiki links open valid `ddowiki.com` pages.

### U5. Update derived counts / self-check and verify derived tables

**Goal:** Keep the data self-check honest and confirm no derived structure went
stale.
**Dependencies:** U1–U4.
**Files:** `index.html` (data self-check block; verify `SAGA_BY_ID` and the
`QUEST_DETAILS` normalized/exact maps if they are build-time constants rather than
runtime-derived).
**Approach:** Container count 58 → **60**; quest-slot count 517 → **541** (+24 =
12 quests × 2 containers). If `SAGA_BY_ID` / detail lookup maps are built at
runtime from `SAGAS`/`QUEST_DETAILS`, no manual edit is needed — **verify** this
rather than assume. Update any hardcoded tier-group counts or the Resume-prompt
"58 containers / 517 quest slots" line if the self-check reads from a constant.
**Test scenarios:**
- Data self-check passes with new totals (no console warning).
- `node --check` (or equivalent JS syntax validation) passes on the built file.

### U6. Live re-source pass — close preview-data gaps

**Goal:** Replace preview placeholders with live-wiki content once available.
**Requirements:** KTD5.
**Dependencies:** U2–U4 (edits the records they create).
**Files:** `index.html` (the 4 placeholder `story` fields, 2 `region`/location
notes, and the Tower of Vengeance giver spelling).
**Approach:** Via Chrome MCP against the **live** (de-Lamannia'd) wiki:
(1) replace the 4 placeholder synopses with real in-game story text; (2) resolve
the 2 placeholder locations (`[1]*` — Hideous to Behold, For Want of a Heart);
(3) confirm "Grin Ousttyl" vs "Grin Ousstyl"; (4) remove the `TODO(u81-resource)`
markers as each is closed. Defer the 13th side quest + raid ("Raiding the
Raiders") to a future update — out of scope here.
**Execution note:** This unit may run in a later session; U1–U5 ship a complete,
usable build without it. Track remaining `TODO(u81-resource)` markers as the
completion signal.
**Test scenarios:** zero `TODO(u81-resource)` markers remain; all 12 stories are
real content; both placeholder locations resolved.

---

## Scope Boundaries

**In scope:** 2 containers, 12 quests, 3 arcs each, full per-quest detail, count
bump, gap-flagging + re-source.

### Deferred to Follow-Up Work
- **13th side quest** (unreleased; debuts 2026-08-19) — add when live.
- **Raid "Raiding the Raiders"** (unreleased) — the app tracks saga quests, not
  raids; revisit when it ships and if raids are ever modeled.
- **Named-loot / augment data** — the tracker does not model loot; explicitly out
  of scope (captured in the dossier only as incidental context).

---

## Risks & Dependencies

- **R1 — Preview (Lamannia) data drift.** Live values may differ from the
  captured preview data (givers, locations, story). *Mitigation:* U6 re-source
  pass; every record carries its source URL for quick re-verification; the 4
  known placeholders are already flagged rather than fabricated.
- **R2 — Legendary fallback miss.** If `normalizeQuestName()`/
  `lookupQuestDetails()` fails to resolve a name, Legendary detail panels go
  blank. *Mitigation:* U4 execution-note explicitly tests 12-for-12 legendary
  resolution before shipping; names are byte-identical across tiers, minimizing
  risk.
- **R3 — Stale derived counts.** A hardcoded self-check or Resume-prompt count
  left at 58/517 would misreport. *Mitigation:* U5 updates counts and verifies
  whether derived tables are runtime-built.
- **R4 — Apostrophe/quoting in names.** "If It Wasn't For Bad Luck",
  "Graz'zt", "Zuggtmoy's Mystic Mushroom", "Araumycos's Domain" contain
  apostrophes. *Mitigation:* use consistent JS string quoting (existing data uses
  double-quoted keys with `'` inside); `node --check` in U5 catches breakage.

---

## Verification / Definition of Done

- `h-demogorgon` and `l-demogorgon` render in the Heroic and Legendary groups
  with 12 quests × 3 arcs each; container count reads **60**, quest slots **541**.
- All 12 Heroic **and** all 12 Legendary quest detail panels populate (Legendary
  via fallback) — no blank panels.
- Per-character progress persists across reload; **schema stays v14**,
  `STORAGE_KEY` unchanged; `ensureShape()` untouched (no new user-editable
  fields).
- Data self-check passes; `node --check` passes on the built HTML.
- Browser visual pass via localhost http server + Claude-in-Chrome (per project
  verification memory).
- Every new record carries a `ddowiki.com` source/wikiUrl; the only invented text
  is `TODO(u81-resource)` markers, each with its source URL, tracked for U6.
- Build gets a fresh `mmddyyyy.x` version stamp and is parked per the
  garage copy-first / 3-build retention workflow.
