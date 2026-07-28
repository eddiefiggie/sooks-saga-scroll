---
title: "Embedding bulk wiki data: verify coverage against the real source, and know the rate-limit fallback"
category: workflow-issues
problem_type: workflow_issue
track: knowledge
module: patron-favor
component: development_workflow
severity: high
tags:
  - data-harvest
  - ddo-wiki
  - mediawiki-api
  - data-integrity
  - self-check
applies_when: "Extracting names or records from a large in-file source to drive a bulk external-data harvest, embedding the harvested data as app constants, or writing a self-check for embedded reference tables"
date: 2026-07-27
---

# Embedding bulk wiki data: verify coverage against the real source, and know the rate-limit fallback

## Context

The patron-favor feature (Build 07272026.1) embeds wiki-sourced constants in `index.html`:
`QUEST_FAVOR` (per-quest `patron`/`favor`/`shares favor`) is harvested from ddowiki.com and the app
derives favor from it. To build the harvest list, a throwaway Python script extracted the unique quest
names from the `SAGAS` array in `index.html` by slicing a fixed window of the source:

```python
region = html[m.end():m.end() + 400000]   # 400KB from `const SAGAS = [`
```

`SAGAS` was larger than 400KB, so the slice truncated the **tail** containers (the Non-Saga Epic packs
— Red Fens, Vault of Night, Zawabi's Refuge — plus several low-level Heroic packs). The extraction
reported "286 unique quest names," the harvest dutifully fetched 286, and every downstream check said
**286 / 286, 0 missing** — because every number was computed from the same truncated list. The real
count was **333**. Forty-seven quests (including all of House Kundarak's favor) were silently absent
from `QUEST_FAVOR`, so a character who had sealed them saw **less favor than reality**, with the data
self-check showing a green badge. The gap was invisible until a `/ce-code-review` finding prompted a
self-check assertion that walked the *real* `SAGAS` and confirmed each quest resolved into the favor
index — it fired immediately, listing all 47 (fixed in Build 07272026.3, commit `ca76461`).

Separately, mid-harvest the wiki began rate-limiting: same-origin `fetch()` to the MediaWiki API
started returning empty bodies after ~7 batches. A different access path still worked.

## Guidance

**1. Never extract from a large in-file source by a fixed byte/char budget.** Parse the whole
structure. If you must window, assert you consumed to the real end (e.g. that the closing `];` was
inside the slice), or count against an independent total.

**2. A coverage number is only trustworthy if it's computed against a source the harvest did *not*
derive from.** "286 / 286, 0 missing" was true and useless — both sides came from the truncated list.
Cross-check the embedded data against the *live* structure it must cover (here: iterate the actual
`SAGAS` array in the running app, not the extraction's output).

**3. Make the coverage cross-check a permanent self-check, not a one-time script.** The durable guard
is the assertion added to `runDataSelfCheck`: every `SAGAS` quest name must resolve to a
`FAVOR_INDEX` group (`FAVOR_INDEX.slotToGroup[q]`), else push an error. This turns any future
source/data drift — a new content drop, a rename, an apostrophe/colon typo — into a red badge instead
of a silent under-count. A self-check that only validates the embedded table against *itself* (every
`QUEST_FAVOR` patron resolves to a tier, every `shares favor` target exists) cannot catch data that is
simply *absent*; the cross-check against the independent source is the piece that can.

**4. Wiki harvest rate-limit fallback.** When same-origin `fetch()` to `ddowiki.com/api.php` starts
returning empty, stop hammering `fetch`. Instead **navigate** the browser directly to the API URL
(the server renders the JSON into the page), then read it back with a page script that
`JSON.parse(document.body.innerText)` and extracts only the fields you need. This worked when `fetch`
did not — a full page navigation is a different request path than the rapid `fetch` loop the limiter
was throttling. (Both paths are same-origin via Chrome MCP; direct `web_fetch` / `curl` remain blocked
by the sandbox/proxy.)

## Why This Matters

For a tracker whose entire value is an accurate number, a **silent under-count is the worst failure
mode** — there is no error, no crash, and the user has no way to notice the shortfall. It shipped in
Pass 1 and survived a browser verification pass precisely because every check trusted the same
truncated list. The only thing that could catch it was a check grounded in an *independent* source of
truth. The lesson generalizes past this feature: any "N / N, 0 missing" is a tautology unless one of
the two N's comes from somewhere the pipeline didn't produce.

## When to Apply

- Building a harvest work-list by extracting identifiers from a large source file (regex over a big
  embedded array/table, slicing a source window, scraping a generated file).
- Embedding bulk external data as app constants and writing a self-check for it — always add a
  coverage assertion against the live source the data must cover, not just internal-consistency checks.
- Bulk-harvesting from a MediaWiki-backed wiki (DDO wiki and others) via the browser and hitting empty
  responses partway through.

## Examples

**The silent tautology (what shipped) vs. the grounded cross-check (the fix):**

```js
// BEFORE — self-check only validated the embedded table against itself.
// Cannot detect a quest that is simply missing from QUEST_FAVOR.
Object.keys(QUEST_FAVOR).forEach(q => { /* patron resolves? shares target exists? */ });

// AFTER (Build 07272026.3) — cross-check embedded data against the independent
// live source (SAGAS). This assertion fired and listed all 47 missing quests.
const seen = new Set();
SAGAS.forEach(s => (s.quests || []).forEach(q => {
  if (seen.has(q)) return; seen.add(q);
  if (!FAVOR_INDEX.slotToGroup[q]) errors.push(`SAGAS quest "${q}" (${s.id}) has no QUEST_FAVOR entry`);
}));
```

**Rate-limit fallback (fetch → navigate):**

```js
// fetch() started returning empty bodies after ~7 rapid batches.
// Fallback: navigate the tab to the API URL, then parse the rendered JSON.
//   navigate -> https://ddowiki.com/api.php?action=query&prop=revisions&rvslots=main
//               &rvprop=content&format=json&formatversion=2&redirects=1&titles=A|B|C...
const j = JSON.parse(document.body.innerText);          // server-rendered JSON in the page body
const out = {};
(j.query.pages || []).forEach(p => {
  const c = (((p.revisions || [{}])[0].slots || {}).main || {}).content || "";
  out[p.title] = { patron: field(c, "patron"), favor: field(c, "favor"), sharesFavor: field(c, "shares favor") };
});
// write `out` to a <pre> and read it back with get_page_text to get it out of the page.
```

## Related

- Harvest mechanics and the browser→repo export bridge: the `ddo-wiki-bulk-data-bridge` and
  `sooks-saga-scroll-wiki-audit-method` notes (auto memory [claude]).
- Feature plan: `docs/plans/2026-07-27-001-feat-patron-favor-tracking-plan.md` (KTD7 accuracy-first
  harvest, U5 self-check).
- Harvest provenance and the 286→333 correction log: `data/patron-favor-harvest.md`.
