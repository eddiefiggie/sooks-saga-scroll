# Patron Favor Harvest — Reconciliation Log

Sourced from the DDO Wiki 2026-07-27 via same-origin MediaWiki API (`action=query&prop=revisions`, per-quest `{{Quest}}` infobox), the accuracy-first path (KTD7). Values are the infobox `favor` (Elite/max) and `patron`, plus `shares favor`. This is the U1 deliverable for accuracy review before embedding into `index.html`.

## Coverage

- **286 / 286** unique tracked quest names resolved, **0 missing**.
- **18 distinct patrons** appear across tracked quests (of 21 total factions; untracked factions like House Kundarak have no quests in the tracked sagas).
- **3 null-patron quests** (no patron / no favor): `Time Flies`, `Turn the Page`, `Vecna Unleashed`.

### Redirects (app quest name → wiki page title)

- `A Break in the Ice` → `A Break In the Ice`
- `A Mad Tea Party` → `Mad Tea Party`
- `Ghost of a Chance (Epic)` → `Ghost of a Chance (epic)`
- `Prove Your Worth (Epic)` → `Prove Your Worth (epic)`

## Favor-groups (shares-favor, counted ONCE)

All 8 groups verified internally consistent (same patron + same favor across members). Members' Elite favor is counted once per group at runtime.

| Favor-group members | Patron | Elite favor |
|---|---|---|
| A Cabal for One + Return to Cabal for One | Agents of Argonnessen | 6 |
| A Legend Revisited + The Legend of Two-Toed Tobias | The Free Agents | 4 |
| Ghost of a Chance + Ghost of a Chance (Epic) | The Free Agents | 4 |
| Gianthold Tor + Return to Gianthold Tor | Agents of Argonnessen | 7 |
| Madstone Crater + Return to Madstone Crater | Agents of Argonnessen | 8 |
| Old Grey Garl + Old Tomb, New Tenants | The Free Agents | 5 |
| Prove Your Worth + Prove Your Worth (Epic) | The Free Agents | 4 |
| Return to Prison of the Planes + The Prison of the Planes | Agents of Argonnessen | 6 |

**Note:** 2 of these groups — `A Legend Revisited`+`The Legend of Two-Toed Tobias` and `Old Grey Garl`+`Old Tomb, New Tenants` — do NOT share a normalized name, so `normalizeQuestName` alone would miss them. The wiki `shares favor` field is authoritative (KTD3).

## Patron reconciliation vs app data

79 quests differ between the harvested wiki patron and the app's existing container/override patron. On inspection, **all** are cases where the app's value is a non-canonical container label (`Various`, `Free Pack`, `X / Y`, `(partial)`) or absent — NOT a genuine per-quest patron. Per KTD7 the wiki per-quest patron wins. No divergence indicates a bad harvest. Examples: `Irestone Inlet` wiki=The Coin Lords vs app='House Deneith (partial)'; `Creeping Death` wiki=The Twelve vs app='The Twelve / House Deneith'; `Astral Ambush`/`Pilgrims' Peril` wiki=The Twelve vs app='Free Pack'.

## Patron → tracked-quest counts

| Patron | Tracked quests |
|---|---|
| The Gatekeepers | 54 |
| The Harpers | 25 |
| Morgrave University | 24 |
| Purple Dragon Knights | 23 |
| Sharn City Council | 23 |
| Agents of Argonnessen | 22 |
| The Summer Court | 15 |
| The Free Agents | 14 |
| Keepers of Lamordia | 14 |
| Cormanthor Elves | 13 |
| Battlehammer Expedition | 12 |
| Keepers of Barovia | 12 |
| The Coin Lords | 8 |
| House Deneith | 6 |
| The Twelve | 5 |
| House Jorasco | 5 |
| House Phiarlan | 4 |
| The Silver Flame | 4 |

## Remaining U1 sub-task

- **PATRON_TIERS** (per-patron favor thresholds + rank names + full reward text) still to be transcribed from the single `/page/Favor` page (already captured this session) into a structured constant. Straightforward transcription; no additional harvest needed.
- Then embed `QUEST_FAVOR` + `PATRON_TIERS` as `index.html` constants (U1 tail) and proceed to U2–U5.
