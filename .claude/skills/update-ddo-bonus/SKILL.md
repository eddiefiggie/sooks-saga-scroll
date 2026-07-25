---
name: update-ddo-bonus
description: Update the Sook's Saga Scroll "Bonus Days" banner when a new DDO bonus event drops. Use whenever the user mentions a new/updated/expired DDO bonus day, a new bonus on the DDO official site, or asks to refresh/clear the bonus banner. This edits an external GitHub gist that the app reads live — it does NOT touch the HTML build, restamp a version, or commit anything to the repo.
---

# Update the DDO Bonus Days banner

The Bonus Days banner in Sook's Saga Scroll is **not baked into the build**. The app
(`refreshBonusBanner()`) fetches a tiny JSON file from a GitHub gist on **every page load**
(`cache: no-store`) and shows a gold band under the subtitle while the bonus is unexpired.
Updating the bonus = editing that gist. It never touches the HTML, never needs a new build,
and is never committed to the repo. The banner self-hides after the expiry date.

## The gist (the only thing you edit)

- **Raw URL (what the app fetches):**
  `https://gist.githubusercontent.com/eddiefiggie/0cf713ba2124e6bf1afb0b0e99ab3702/raw/ddo-bonus.json`
- **Gist ID:** `0cf713ba2124e6bf1afb0b0e99ab3702`  ·  **File:** `ddo-bonus.json`
- Owner `eddiefiggie`; edit with the `gh` CLI (needs `gist` scope — already present).

## JSON contract (must be VALID JSON — no trailing commas)

The banner parser requires `headline` and `expires`; `details` and `updated` are optional.
Anything malformed, missing a required field, or past `expires` → **no banner** (the app logs
why to the console; the app is otherwise unaffected).

```json
{
  "headline": "+10% Reaper XP",
  "expires": "2026-07-26",
  "details": "All players — no VIP required.",
  "updated": "Jul 23"
}
```

| Field      | Req | Meaning / format |
|------------|-----|------------------|
| `headline` | ✅  | The bonus itself, short. e.g. `"+10% Reaper XP"`, `"Double Mysterious Remnants"`. Renders bold. |
| `expires`  | ✅  | `YYYY-MM-DD`. Banner shows through the **end of that local day**, then hides. Use the event's last day. |
| `details`  | –   | One short clarifier line, smaller text. Omit if the headline says it all. |
| `updated`  | –   | Short "as of" stamp shown as `· as of <value>`, e.g. `"Jul 23"`. Set to the date you edit. |

Banner renders as: `⚑ Bonus Days · <headline> · <details> · thru <Mon D> · as of <updated>`.

> ⚠️ A past bug: the gist once held `{"headline":"No current bonus day.",}` — invalid (trailing
> comma) **and** missing `expires`. Always emit real, validated JSON.

## Runbook

### 1. Find the current bonus on the DDO official site
DDO bonus posts follow the URL pattern `https://www.ddo.com/news/ddo-bonus-MMDDYY` (US date of
the post). Probe today's date first, then walk back a few days:

```bash
for d in $(python3 -c "import datetime as D;print(*[(D.date.today()-D.timedelta(n)).strftime('%m%d%y') for n in range(7)])"); do
  code=$(curl -s -o /dev/null -w "%{http_code}" "https://www.ddo.com/news/ddo-bonus-$d")
  echo "ddo-bonus-$d -> $code"
done
```
`200` = a live post exists at that URL; `302` = redirect (no post / expired slug). Take the most
recent `200`.

### 2. Read the bonus — from the META tags, not the rendered body
The news page body is JS-rendered and **absent from the static HTML**, so don't trust a scrape of
the visible text (and don't let a summarizer fill gaps). The authoritative summary is in the page's
meta description:

```bash
curl -s "https://www.ddo.com/news/ddo-bonus-MMDDYY" \
  | grep -io '<meta[^>]*\(name="description"\|og:description\)[^>]*content="[^"]*"'
```
e.g. `content="DDO Bonus Days bring you +10% Reaper XP, now through July 26th!"` →
headline `+10% Reaper XP`, expires `2026-07-26`. If the meta is ambiguous, confirm the % and dates
with a WebFetch of the same URL, but the meta description wins.

### 3. Write valid JSON and PATCH the gist
Build the payload with Python (guarantees valid JSON), then patch:

```bash
python3 - <<'PY'
import json
bonus = {"headline":"+10% Reaper XP","expires":"2026-07-26",
         "details":"All players — no VIP required.","updated":"Jul 23"}
content = json.dumps(bonus, indent=2, ensure_ascii=False) + "\n"
json.dump({"files":{"ddo-bonus.json":{"content":content}}}, open("/tmp/gist_patch.json","w"), ensure_ascii=False)
PY
gh api -X PATCH gists/0cf713ba2124e6bf1afb0b0e99ab3702 --input /tmp/gist_patch.json \
  --jq '.files["ddo-bonus.json"].content'
```

### 4. Verify it's live (cache-bust the raw URL)
```bash
curl -s "https://gist.githubusercontent.com/eddiefiggie/0cf713ba2124e6bf1afb0b0e99ab3702/raw/ddo-bonus.json?nocache=$(python3 -c 'import time;print(int(time.time()))')"
```
Confirm the JSON matches what you wrote. The banner appears on the next page load.

## Clearing the banner (no current bonus)
To take the banner down before its natural expiry, PATCH the gist with an already-past `expires`:
```json
{ "headline": "No current bonus day.", "expires": "2000-01-01" }
```
(A past `expires` hides it; still emit valid JSON.)

## Do NOT
- ❌ Restamp / create a new build (`sooks-saga-scroll-MMDDYYYY-N.html`) — the HTML is unchanged.
- ❌ Edit `index.html` or the parked build.
- ❌ `git commit` — the gist is external state; the repo intentionally keeps no record of the bonus.
- ❌ Trust the JS-rendered article body over the meta description.
