---
title: A CSS content hex-escape silently eats the space that follows it
module: sooks-saga-scroll
date: 2026-07-26
problem_type: logic_error
component: frontend_css
severity: low
symptoms:
  - "A CSS ::after ribbon set to content: \"\\2726 BANKED\" rendered as \"✦BANKED\" with no gap between the glyph and the word"
  - "The space typed after a \\XXXX unicode escape in a CSS content string disappears from the rendered text"
root_cause: logic_error
resolution_type: code_fix
tags:
  - css
  - pseudo-element
  - unicode-escape
  - content
  - gotcha
---

# A CSS content hex-escape silently eats the space that follows it

## Problem

Build 07262026.3 added a "✦ BANKED" corner ribbon via `.saga.banked::after { content: "\2726 BANKED" }`. It rendered as **"✦BANKED"** — the glyph and the label with no gap — even though a space was typed between them.

## Symptoms

- The rendered pseudo-element text is missing the space that clearly appears in the source string, immediately after a `\XXXX` hex escape.
- In-browser, `getComputedStyle(el, '::after').content` came back as `"✦BANKED"`, not `"✦ BANKED"`.

## Solution

Use a non-breaking-space escape (`\00a0`) — or two spaces — after the hex escape, so a visible gap survives:

```css
/* before — the space is consumed as the escape terminator: */
content: "\2726 BANKED";   /* → "✦BANKED" */

/* after — \00a0 is an explicit non-breaking space: */
content: "\2726\00a0 BANKED";   /* → "✦ BANKED" */
```

## Why This Works

CSS unicode escapes are `\` + up to six hex digits. The parser reads hex digits greedily until it hits a non-hex character; **a single whitespace after the escape is treated as the terminator of the escape and is discarded**, not rendered. So `"\2726 BANKED"` is parsed as `✦` + `BANKED` with the terminator space swallowed. Encoding the gap as its own character (`\00a0`, a non-breaking space) — or adding a second literal space (the first ends the escape, the second renders) — puts a real space into the string.

## Prevention

- When a CSS `content` string is `<hex-escape><space><text>`, encode the gap as `\00a0` (or `\0020`), not a bare space. This project uses glyph-plus-label pseudo-elements often (`✦`, `☠`, stars); the pattern recurs.
- Verify pseudo-element text in-browser with `getComputedStyle(el, '::after').content` rather than by eye in the source — the swallowed space is invisible in the CSS but obvious in the computed value.
