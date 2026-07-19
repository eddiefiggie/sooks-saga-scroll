# Concepts

Shared domain vocabulary for this project — entities, named processes, and status concepts with project-specific meaning. Seeded with core domain vocabulary, then accretes as ce-compound and ce-compound-refresh process learnings; direct edits are fine. Glossary only, not a spec or catch-all.

## Containers & progress

### Saga container
A grouping of DDO quests that belong to a named saga chain, tracked as one unit for quest completion and its saga reward. Each container rolls up two independent progressions: how many of its quests are completed for the saga reward, and how many are First-Time Reaper.

### Non-Saga container
A container that groups standalone quests which are *not* part of any saga chain, kept only for leveling and First-Time Reaper tracking. Non-Saga containers are excluded from the "sagas complete" and "rewards banked" tallies and from the tier breakdown — they are a convenience grouping, not a saga.

### First-Time Reaper (FTR)
The state of having completed a quest on Reaper difficulty for the first time. Tracked per quest and rolled up per container as the count of reaper-eligible quests still unreaped; a container is "sealed" when none remain. FTR is independent of ordinary quest completion — completing a quest on non-Reaper difficulty never advances it.

FTR is a **permanent, one-time claim**: once earned it is never lost and it does not re-trigger. This sets it apart from the ordinary completion seal, which is per-cycle and *is* cleared when a saga reward is redeemed and the saga is re-run. Because the two can diverge — a quest can be reaped but not currently completed — any path that marks a quest must never infer "first time" from the completion state, and must not re-announce or re-seal FTR when re-marking a quest whose FTR is already claimed.

### Reaper-eligible quest
A quest that can count toward a container's First-Time Reaper progress. Some quests are permanently excluded (solo-only / no-Reaper quests) and never count toward the reaper denominator, so a container of only these has zero reaper-eligible quests.

## Layout

### Corner cluster
A cluster of live-status cards anchored to a screen corner: one cluster holds the character roster and the Reaper LFM alert cards; the other holds the server-population bar and the online-guild roster. Fixed to the viewport at desktop width; at narrow widths the clusters leave their fixed corners and reflow into the document.

### Roll-over tier
One of the three priority groups a saga container header or quest row is composed of for responsive stacking: Tier 1 (name + level), Tier 2 (the actions group — reward pill + progress bars on a container header, or the completion controls on a quest row), and Tier 3 (LFM info). As a row loses width the tiers roll to new lines lowest-priority first (LFM, then the actions group), and each tier rolls as one atomic unit — a tier never fragments so a lone bar or button never strands on its own line.
