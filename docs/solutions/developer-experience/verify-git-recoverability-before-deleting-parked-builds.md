---
title: Verify git-recoverability before bulk-deleting a "safe" local build cache
date: 2026-07-26
category: developer-experience
module: sooks-saga-scroll
problem_type: workflow_issue
component: development_workflow
severity: medium
applies_when:
  - "About to delete a gitignored 'rollback' / 'backup' / 'to-delete' folder you did not create this session"
  - "Cleaning up pruned parked builds under the garage 3-file retention convention"
  - "A folder's stated purpose implies its contents are redundant with a source of truth (git, a remote, another copy)"
tags: [git, cleanup, data-loss, parked-builds, gitignore, retention, safety]
---

# Verify git-recoverability before bulk-deleting a "safe" local build cache

## Context

This project ships as a single self-contained HTML file, versioned by copy: each
build is `sooks-saga-scroll-<MMDDYYYY>-<N>.html`, and the garage convention keeps
a **3-file retention** on disk (the two most recent overall plus the most-recent
previous-day build as a cross-day rollback anchor). Older builds are pruned. For
a stretch of the project's history, "pruned" meant *moved aside* into a
gitignored `_to_delete/` folder — `.gitignore` described it as **"Pruned builds
kept locally for rollback, not published."**

During a routine "clean up loose ends" pass, `_to_delete/` had accumulated 17
old builds (~23 MB). The obvious move was `rm -rf _to_delete/` — the folder's
name and its `.gitignore` comment both say it is throwaway rollback storage, so
its contents *should* all be reproducible from git history. A recoverability
check before deleting proved that assumption wrong: **5 of the 17 builds existed
only in `_to_delete/` and were in no git commit on any branch.** They were
intermediate builds created and superseded within a single session, before any
of them was ever committed as the "live build." Deleting the folder blind would
have permanently destroyed those 5 files, with no git recovery path.

## Guidance

**Before bulk-deleting any folder whose stated purpose implies its contents are
redundant with a source of truth, verify that redundancy per file — don't trust
the label.** A `_to_delete/` folder, a `.bak` pile, a `backup` dir, or anything
named "rollback" is a *claim* of safety, not a guarantee. A gitignored folder is doubly suspect: being
ignored, its files were never eligible to be committed, so it can hold the only
copy of something that never made it into history.

For git-tracked build artifacts, build the set of every artifact path ever
committed on any branch, then diff the folder against it and surface any
only-local files for an explicit decision before deleting:

```sh
# every build-file path ever committed, on any branch
git rev-list --all --objects | awk '{print $2}' \
  | grep 'sooks-saga-scroll-.*\.html' | sort -u > /tmp/in-history.txt

# classify each file in the folder: recoverable vs only-local
for f in _to_delete/*.html; do
  b=$(basename "$f")
  if grep -qx "$b" /tmp/in-history.txt; then
    echo "recoverable  $b"
  else
    echo "ONLY-LOCAL   $b"   # deleting this is irreversible
  fi
done
```

If the classification shows any `ONLY-LOCAL` files, treat the deletion as
irreversible: surface the specific files and confirm intent before proceeding,
rather than letting a folder-level `rm -rf` decide their fate silently. (In this
project the user was shown the 5 only-local builds and chose to delete the whole
folder anyway — an informed, explicit decision, which is exactly the point:
irreversible loss should be a choice, not a side effect.)

The durable fix for the recurring case is to stop the parallel cache from
drifting: prune by plain `rm` of the *tracked* build path (so git history is the
single rollback source and every pruned build is genuinely recoverable), instead
of moving files into a gitignored side folder that can accumulate never-committed
copies.

## Why This Matters

The retention convention's safety rests on one assumption: "git history holds
everything important, so the local `_to_delete/` cache is pure convenience." That
assumption silently fails for any build that is created and superseded *within a
single session* before a commit — the exact builds that get pruned fastest. The
folder's name and `.gitignore` comment actively reinforce the false belief that
the contents are disposable. The general trap is broader than this repo: **a
directory's documented purpose is a description of intent, not proof of its
contents' recoverability**, and `.gitignore` status makes "it's all in git" less
likely to be true, not more.

The cost of the check is one throwaway `git rev-list` diff; the cost of skipping
it is unrecoverable data loss the folder's own labeling told you was impossible.

## When to Apply

- Any bulk delete (`rm -rf`) of a local folder you did not create this session —
  especially a `_to_delete/` folder or anything named like a "backup", "bak", or
  "rollback" store, or one that is gitignored.
- Pruning parked builds in this project or any copy-versioned single-file build
  workflow, where superseded intermediate builds may never have been committed.
- Any time a folder's description asserts its contents are redundant with a
  source of truth — verify the redundancy for each file before trusting it.

## Examples

The check that caught the loss in this session (17 files in `_to_delete/`, 5
only-local):

```
✓ in git history        sooks-saga-scroll-07182026-9.html
⚠ ONLY in _to_delete/   sooks-saga-scroll-07142026-2.html
⚠ ONLY in _to_delete/   sooks-saga-scroll-07182026-8.html
⚠ ONLY in _to_delete/   sooks-saga-scroll-07192026-2.html
⚠ ONLY in _to_delete/   sooks-saga-scroll-07192026-3.html
⚠ ONLY in _to_delete/   sooks-saga-scroll-07252026-1.html
... (12 others all present in git history)
```

Contrast the two prune styles:

```sh
# DRIFT-PRONE: moves the file out of git's reach; the side folder can end up
# holding builds that were never committed (the only copy).
mv sooks-saga-scroll-07262026-6.html _to_delete/

# SAFE: deletes the tracked path; git history is the single, complete rollback
# source, so every pruned build stays recoverable via `git show <sha>:<path>`.
rm sooks-saga-scroll-07262026-6.html
git add -A sooks-saga-scroll-07262026-6.html   # records the deletion
```

## Related

- The 3-file retention convention and the park/prune workflow are documented in
  the project `README.md` (the "Currently parked" park-line block).
- Sibling garage-workflow practice: `verify-single-file-html-without-node.md`
  (how a parked build is verified before it ships).
