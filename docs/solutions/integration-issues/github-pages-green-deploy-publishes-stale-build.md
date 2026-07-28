---
title: "A green GitHub Pages deploy can still publish a stale build"
category: integration-issues
problem_type: integration_issue
track: bug
module: deploy
component: development_workflow
severity: high
root_cause: config_error
resolution_type: config_change
symptoms:
  - "The 'Deploy to Pages' workflow completes green on every push, but the live site keeps serving an old build"
  - "curl to the live URL returns HTTP 200 with a stale build stamp while the repo's index.html is a newer build"
  - "A shipped feature is absent from the live site despite several successful deploys"
date: 2026-07-27
---

# A green GitHub Pages deploy can still publish a stale build

## Problem

The app's GitHub Pages workflow (`.github/workflows/deploy.yml`) publishes whichever
date-stamped build filename it finds in `README.md`. It read the **first**
`sooks-saga-scroll-*.html` filename *anywhere* in the README via `grep -m1`, and that
first match was a stale reference near the top of the file. Three successive builds
(`07272026.1`, `.2`, and `.3`) each "deployed successfully" but re-published the **old** pre-feature
build — the new Patrons feature was never actually live, and nobody noticed because every
deploy was green.

## Symptoms

- `gh run list` shows `Deploy to Pages … completed … success` for the new build's commit.
- The live site (`https://eddiefiggie.github.io/sooks-saga-scroll/`) returns **HTTP 200**
  but serves the old build: `class="build">Build 07262026.8` while the repo's `index.html`
  is `Build 07272026.3`, and the favor feature markers (`QUEST_FAVOR`, `renderPatronsView`)
  are absent from the served HTML.
- File size of the served page matches the old build, not the (larger) new one.

## What Didn't Work

- **Trusting the workflow's "success" status.** Every deploy was green; the workflow ran to
  completion. Success only meant "the job that copies *a* file finished," not "the file you
  intended is live." The whole failure hid behind a passing status.
- **Updating the wrong README line.** Each build I updated the *Files*-section "live build"
  line to the new filename — but that line is lower in the README. The workflow's
  `grep -m1` takes the first filename match in the *entire* file, which was a stale
  reference in the intro blockquote (`sooks-saga-scroll-07262026-8.html`). Updating a
  different line had no effect on what shipped.

## Solution

Two fixes — one to the missed process step, one to make the workflow robust.

**1. Update the canonical marker line** the workflow actually reads. In `README.md` the
intro carries a `**Currently parked:**` line; that (as the first filename in the file) is
what `grep -m1` resolves. It must be bumped every park:

```
> **Currently parked:** `sooks-saga-scroll-07272026-4.html` — Build `07272026.4` …
```

**2. Anchor the workflow on that marker, not "first filename anywhere"**
(`.github/workflows/deploy.yml`):

```bash
# BEFORE — grabs the first sooks-saga-scroll-*.html anywhere in the README,
# so any earlier stale mention silently hijacks the publish.
LIVE=$(grep -m1 -oE 'sooks-saga-scroll-[0-9]{8}-[0-9]+\.html' README.md)

# AFTER — read the filename ONLY from the canonical "Currently parked:" line.
LIVE=$(grep -m1 'Currently parked:' README.md | grep -oE 'sooks-saga-scroll-[0-9]{8}-[0-9]+\.html' | head -1)
```

## Why This Works

The bug was a mismatch between the *intended* source of truth (the "Currently parked:" line)
and the workflow's *actual* selector (first filename anywhere). Any older filename appearing
earlier in the growing README — an intro reference, a changelog note — silently won. Scoping
the grep to the `Currently parked:` line makes the selector match the intent, so a stray
mention elsewhere can no longer hijack the publish, and the loud `::error::` on an
unresolved/absent file surfaces a genuinely missed update instead of silently shipping the
wrong build. (Fixed in commit `b31e94a`.)

## Prevention

- **Verify the live output, never the pipeline status.** A green CI/deploy job means the job
  ran, not that the correct artifact is live. After every deploy, assert the *served* content:

  ```bash
  curl -s "https://eddiefiggie.github.io/sooks-saga-scroll/index.html?cb=$RANDOM" \
    | grep -o 'class="build">Build [0-9.]*'   # must equal the build you just parked
  ```

  **HTTP 200 alone is worthless here — the stale build returns 200 too.** Grep for a version
  stamp or a feature marker that only exists in the new build.
- **Make "success measured against the wrong source" impossible.** This is the same failure
  shape as a data check that validates output against the same truncated input it produced:
  the check passes and proves nothing. Anchor verification on an *independent* signal (the
  live page's own stamp), not on the process that produced the artifact.
- **When a deploy reads a value from a doc, key it on a specific marker string**, not
  positional/first-match parsing of a file that will accumulate other matches over time.
