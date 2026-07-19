---
title: "Verifying single-file HTML builds: node --check + a real-browser pass (jsc is the fallback)"
module: sooks-saga-scroll
date: 2026-07-18
last_updated: 2026-07-18
problem_type: developer_experience
component: development_workflow
severity: medium
applies_when:
  - "Editing a single-file HTML app's inline <script> and you need a fast syntax gate before parking a build"
  - "You need to confirm a build renders correctly — layout, CSS, responsive behavior — not just that the JS parses"
  - "node/deno/bun are missing (fall back to the jsc syntax check) or browser automation is blocked (defer the visual pass to the user)"
tags:
  - verification
  - node
  - jsc
  - browser
  - container-queries
  - single-file-html
  - offline
---

# Verifying single-file HTML builds: syntax + a real-browser pass

## Context

`sooks-saga-scroll` is a single ~16k-line offline HTML file with a large inline `<script>`. It has no build step and no module boundaries, so one syntax slip anywhere breaks the whole page silently — and a layout/CSS mistake ships looking "done" because it produces no error at all. Both a **syntax gate** and a **visual gate** are needed before parking a build.

Earlier in the 2026-07-18 session both gates were unavailable (no `node` on PATH; Claude-in-Chrome blocked `localhost`), which forced a syntax-only `jsc` workaround and cost a whole dead build (a container-query bug that renders nothing — see [container-query-needs-ancestor-container](../design-patterns/container-query-needs-ancestor-container.md)). Both gates were then restored later the same session: **`node` was installed** and **the Claude-in-Chrome `localhost` permission was granted**. This doc reflects that current, working setup, with the old workaround kept as a fallback.

## Guidance

Use two gates, in order.

### 1. Syntax gate — `node --check` (primary), `jsc` (fallback)

`node` is installed (`/opt/homebrew/bin/node`). Extract the inline script and check it:

```bash
awk '/<script>/{f=1;next} /<\/script>/{f=0} f' build.html > /tmp/app.js
node --check /tmp/app.js && echo "SYNTAX_OK"
```

**Fallback when node is unavailable:** JavaScriptCore's CLI (`jsc`) ships on every macOS and compiles the source *without executing DOM code* via the `Function` constructor — it catches the same syntax errors (`unbalanced braces/parens, broken template literals, stray backticks`) as `node --check`:

```bash
JSC=/System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Helpers/jsc
"$JSC" -e 'var s=readFile("/tmp/app.js"); try{ new Function(s); print("SYNTAX_OK"); } catch(e){ print("SYNTAX_ERROR: "+e); }'
```

Pair either with a **static grep for dangling references** — after deleting a function, grep that no call sites remain (only comments); after removing an element, grep that nothing still `getElementById`s it and would throw on null.

### 2. Visual gate — load it in a real browser (Claude-in-Chrome)

The syntax gate proves the JS *parses*; it says nothing about whether the page *renders correctly*. Load the build and look:

```bash
python3 -m http.server 8755   # serve the project dir (background); the navigate tool rejects file://
```

Then, via Claude-in-Chrome: `tabs_context_mcp` → `navigate` to `http://localhost:8755/<build>.html` → `read_console_messages` (onlyErrors) for runtime errors → `computer` screenshot / `zoom` to inspect.

Two gotchas specific to this app:

- **It boots to an empty state** (no character) — the saga UI only appears once a character exists. Inject a test one with `javascript_tool`: set `state.activeChar`, push a name into `state.characters`, then build `state.progress[name][sagaId] = {quests, reward, notes, sagaDone, skipDone}` from the `SAGAS` global (mark some quests `"reaper"`, some `sagaDone`, one `reward: "banked"`), and call `renderAll()`. **For UI that depends on *live-fetched* data (not `state`)** — e.g. the LFM tags, which come from a DDO Audit fetch — inject `state` alone won't show them; monkeypatch the global function that supplies them and re-render, e.g. `window.lfmForTier = function(q){ return q==="Some Quest" ? {count:1,skulls:2,leader:"X",party:3,minLevel:3,maxLevel:5,difficulties:new Set(["Reaper"])} : null; }; renderAll();` (top-level `function` declarations are `window` properties in a classic script, so reassigning wins). Clean up after: remove any injected `<style>`, `localStorage.removeItem('sooksSagaScroll')`, stop the server (the monkeypatch is runtime-only and vanishes on reload).
- **The screenshot viewport is fixed (~1567px)**, so `resize_window` alone won't change the rendered layout. To test width-driven behavior (e.g. container-query roll-over), inject a temporary `<style>` that constrains the element's width (e.g. `.scroll{max-width:470px}`) and screenshot that. `javascript_tool` occasionally returns `[BLOCKED: Cookie/query string data]` (a false positive) — the code still ran; verify via a screenshot rather than the return value.

If the browser permission isn't available, **defer the visual pass to the user** — state plainly what was and wasn't machine-verified. That matches the project's established practice of eyeballing each build in real Chromium.

## Why This Matters

**Only the browser catches layout/CSS/visual bugs.** `node --check`, `jsc`, and even a jsdom harness all check JS — none of them render CSS, so none can see a broken layout. This session proved it concretely: a **dead container query** (rendered nothing), a **reward pill misaligned above the bars**, and **dark-on-dark reaper text** all passed the syntax gate cleanly and were caught only by a real-browser screenshot or a code reviewer's reasoning. Treating a green syntax check as "verified" is exactly how a non-functional build ships.

So the syntax gate is necessary but not sufficient: it closes the "typo breaks the whole page" risk cheaply, but the visual gate is what actually confirms the feature works.

## When to Apply

- **Every build**, before parking it in the copy-first + park-retention flow: syntax gate always; visual gate whenever the change touches layout, CSS, or anything user-visible (which for this app is almost everything).
- Lean on the `jsc` fallback only when `node` is genuinely missing, and on user-deferred visual verification only when browser automation is genuinely blocked — both are degraded modes now, not the default.

## Examples

Syntax gate on a build after edits:

```bash
awk '/<script>/{f=1;next} /<\/script>/{f=0} f' sooks-saga-scroll-07182026-7.html > /tmp/app.js
node --check /tmp/app.js && echo SYNTAX_OK
```

Dangling-reference check after deletions (expect matches only in comments):

```bash
grep -n 'renderProgress()\|charSummary(' build.html
grep -n 'getElementById("progressBand")' build.html   # expect: none
```

Visual gate that caught a real bug this session: narrowing `.scroll` to 470px via an injected style, then screenshotting, revealed the saga headers rolling their tiers uniformly (name+level on top, pill+bars rolled below) — confirming the container-query fix that a syntax check could never have validated.
