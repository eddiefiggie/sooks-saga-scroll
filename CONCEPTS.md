# Concepts

Shared domain vocabulary for this project — entities, named processes, and status concepts with project-specific meaning. Seeded with core domain vocabulary, then accretes as ce-compound and ce-compound-refresh process learnings; direct edits are fine. Glossary only, not a spec or catch-all.

## Containers & progress

### Saga container
A grouping of DDO quests that belong to a named saga chain, tracked as one unit for quest completion and its saga reward. Each container rolls up two independent progressions: how many of its quests are completed for the saga reward, and how many are First-Time Reaper.

### Non-Saga container
A container that groups standalone quests which are *not* part of any saga chain, kept only for leveling and First-Time Reaper tracking. Non-Saga containers are excluded from the "sagas complete" and "rewards banked" tallies and from the tier breakdown — they are a convenience grouping, not a saga.

### Tier
The top-level classification of a container that determines both the section it displays under and its level band — one of six ordered values: Heroic, Non-Saga Heroic, Non-Saga Epic, Epic, Legendary, Non-Saga Legendary. Not to be confused with a [Roll-over tier](#roll-over-tier), which is a responsive-layout row group.

Containers render grouped by tier in that fixed section sequence; within a tier they are ordered by character level (ascending by the low end of each container's level range). That within-tier level ordering is maintained by hand in the container list — nothing re-sorts it at display time — so a new container has to be placed at its level-correct spot rather than appended to its tier.

### Quest Completed seal
The master "have I completed this quest" marker, per quest per character, per life
(`sagaDone` in stored state). Set by a quest row's left checkbox and auto-filled by the
Reaper / Skip / Saga-Reward controls; it drives saga-reward auto-banking and is the field
every *derived* per-character status reads — [Patron Favor](#patron-favor) sums it, a raid's
[Keyed](#keyed) status counts it. It **cascades** across same-tier containers that share a
quest (so checking a quest once marks it everywhere it appears), and it is **per-cycle** —
cleared when a saga reward is redeemed and the saga is re-run. Distinct from the raw
"how it was marked" state (Reaper / Skip / plain), which is tracked separately; a derivation
that reads that raw state instead of this seal silently misses the checkbox's completions.

### First-Time Reaper (FTR)
The state of having completed a quest on Reaper difficulty for the first time. Tracked per quest and rolled up per container as the count of reaper-eligible quests still unreaped; a container is "sealed" when none remain. FTR is independent of ordinary quest completion — completing a quest on non-Reaper difficulty never advances it.

FTR is a **permanent, one-time claim**: once earned it is never lost and it does not re-trigger. This sets it apart from the ordinary completion seal, which is per-cycle and *is* cleared when a saga reward is redeemed and the saga is re-run. Because the two can diverge — a quest can be reaped but not currently completed — any path that marks a quest must never infer "first time" from the completion state, and must not re-announce or re-seal FTR when re-marking a quest whose FTR is already claimed.

### Reaper-eligible quest
A quest that can count toward a container's First-Time Reaper progress. Some quests are permanently excluded (solo-only / no-Reaper quests) and never count toward the reaper denominator, so a container of only these has zero reaper-eligible quests.

## Favor

### Patron favor
A character's accumulated favor with a single DDO patron, as tracked by this app. Project-specific
meaning: it is **derived, not stored** — computed live each render as the sum of the Elite base favor
of every quest whose per-life saga seal (`sagaDone`) is set, grouped by the quest's patron. Two
deliberate simplifications define it here: every sealed quest is counted at **Elite** (maximum
favor), and the total is **this-life only** — it follows the saga seal and resets on True
Reincarnation, with no permanent "ever-earned" underlay. This makes it a this-life favor tracker, not
a mirror of the game's permanent patron favor. Distinct from account-wide total favor, which is
explicitly out of scope.

**Raids contribute too, under a different difficulty assumption (Build 07292026.7).** A completed
raid awards favor to its own patron from a wiki-sourced base value (`RAID_FAVOR`, the Elite base),
but — because the app can't know which difficulty was actually run — a plain **Complete** is assumed
run on **Hard** (⅔ of the Elite base, rounded), while using the **Reaper** button is assumed
**Reaper**, which grants the same favor as Elite in DDO (the full base). Unlike quests, a raid's award
is a flat per-raid amount, not a `shares favor` [favor-group](#favor-group) dedup — each raid is one
instance with one patron. The award appears/disappears live with the raid's own `sagaDone`, so
un-completing a raid removes its favor. This is why raids skip the `QUEST_FAVOR` group loop entirely
in `computeCharacterFavor` yet still show up on the Patrons dashboard.

### Favor-group
The unit favor is counted by: a quest together with any `shares favor` partners across its Heroic /
Epic / Legendary versions. Verified on the DDO Wiki — the Heroic and Epic versions of a quest carry
the **same patron and the same favor value** and are linked by a `shares favor` infobox field;
completing either at Elite grants that favor **once**. So favor is deduped per favor-group and never
counted per tier-instance (naive per-slot summing would double-count every Heroic↔Epic pair). The
app's existing `normalizeQuestName` already groups these versions and is the ready-made dedup key,
cross-checked against the wiki's `shares favor` field. A quest may have **no patron and no favor**
(some free-to-play/event quests), represented explicitly rather than defaulted.

## Layout

### Corner cluster
A cluster of live-status cards anchored to a screen corner: one cluster holds the character roster and the Reaper LFM alert cards; the other holds the server-population bar and the online-guild roster. Fixed to the viewport at desktop width; at narrow widths the clusters leave their fixed corners and reflow into the document.

### Roll-over tier
One of the three priority groups a saga container header or quest row is composed of for responsive stacking: Tier 1 (name + level), Tier 2 (the actions group — reward pill + progress bars on a container header, or the completion controls on a quest row), and Tier 3 (LFM info). As a row loses width the tiers roll to new lines lowest-priority first (LFM, then the actions group), and each tier rolls as one atomic unit — a tier never fragments so a lone bar or button never strands on its own line. Distinct from a container's [Tier](#tier) (Heroic/Epic/Legendary classification) — this "tier" is a within-row layout band, not a saga grouping.

## Live data

### Reference cache
The locally cached copy of DDO Audit's static lookup tables — the `areas` table
(area id → name/region) and the `quests` table (quest id → name, area id, raid
flag, adventure pack). Stored in `localStorage` under its own key, separate from
the app's saved `state`, with a multi-day TTL. It is the shared backbone of the
live features: the guild roster's location line, live LFM badges, and quest
Autocomplete all resolve a numeric id (a character's `location_id`, an LFM's
`quest_id`) against it. Distinct from the hand-authored quest/saga data that
renders the containers — a container can display perfectly while the reference
cache is stale, which is why a stale cache breaks the *live* features only.

### Content epoch
An integer stamped into the [Reference cache](#reference-cache) and bumped whenever
a game update adds quests or areas the current build knows about. Read-time the
cache is only trusted when its stored epoch matches the current one; an older or
absent epoch forces a one-time refetch. It is a third invalidation signal
alongside the TTL and the row-shape guard, and the only one that catches
*content-only* upstream changes (new rows, same schema) — the case where a new
expansion's data would otherwise stay invisible until the TTL lapsed.

### Group status (roster: Solo / Grouped / Guild Grouped)
The per-member party status shown on the online-guild roster, read directly and
reliably from each member's live record via `is_in_party` and `group_id` (as of
Build 07262026.4). Three states: **Solo** (`is_in_party` false — not in a party),
**Grouped** (in a party, but no *online guildmate* shares the party), and **Guild
Grouped** (shares a nonzero `group_id` with one or more other online guildmates —
the tooltip names them). Because it uses per-character party status, it sees
private and full groups too, so "Solo" is honest. Earlier builds inferred this
from the advertised-LFM feed (`_groupIndex`), which could not see unadvertised
groups and so used a hedged "Pugging/blank"; that LFM-derived path was removed
once the live probe confirmed `is_in_party`/`group_id` ride the guild fetch. (The
[derived-cache reset parity](docs/solutions/logic-errors/derived-cache-reset-parity.md)
learning came from that now-removed `_groupIndex`; its general lesson still holds.)

### Quest Autocomplete
An opt-in, per-character setting that watches the live location feed and, when the
tracked character enters a quest instance the scroll knows, automatically marks
that quest for them. Two modes: **Non-Reaper** marks only the green Completed seal;
**Reaper** also seals [First-Time Reaper](#first-time-reaper-ftr) — degrading to a
plain Completed mark when FTR is already claimed, since a permanent claim is never
re-celebrated. It resolves *which* container to mark by the character's level band,
so a quest that exists under more than one [Tier](#tier) is marked on the tier
matching the character's current level; because those per-tier copies are tracked
independently, marking or resetting one never affects the other.

### Keyed
A raid's flagged/unlocked state for a character, used in the [Raids](#raids-tab) tab.
Like [Patron Favor](#patron-favor) it is **derived, not stored** — a raid is *keyed*
when all of its prerequisite (flagging) quests are complete for that character. Raids
with no prerequisite quests are always keyed. Because a raid's flagging quests are the
same quest identities that may also live in Saga-tab containers, keyed derivation reads
the same completion state the Saga tab writes (via the same-tier `SHARED_QUESTS`
cascade), so the two tabs never disagree.

### Raids Tab
A per-character tracker for every permanent Heroic/Epic/Legendary raid, modeled like
Saga-tab content ([Tier](#tier) headers → containers → checkable quest rows). Each raid
is a container holding its flagging quests plus the raid instance; within a tier,
[keyed](#keyed)-required raids sort before no-key raids. Seasonal/event raids are out of
scope, and the reaper seal is a saga concept not applied to raids.

## Flagged ambiguities

- **"Tier"** is used for two unrelated things: a container's [Tier](#tier) (its Heroic/Epic/Legendary/Non-Saga classification and level band) and a [Roll-over tier](#roll-over-tier) (one of the three responsive-layout priority bands a row stacks into). When unqualified, "tier" means the container classification; the layout sense is always "roll-over tier".
