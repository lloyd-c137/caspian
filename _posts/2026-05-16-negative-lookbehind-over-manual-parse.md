---
layout: post
title: "When Your Manual Parser Becomes a Regex: A KaTeX Patch Story"
tags: [katex, regex, open-source, refactoring]
---

I was patching KaTeX's `\htmlData` command recently. The bug: values containing commas (like `[a,b]`) would get incorrectly split because the parser called `value.split(",")` — naive and unforgiving.

The original fix I wrote used a manual character loop with an escape flag. It worked, but it was 25 lines of state management for what is essentially a two-regex problem:

```javascript
// Before: 25 lines of for-loop with escaped flag
// After: 2 lines
const data = value.split(/(?<!\\),/)
    .map(s => s.replace(/\\,/g, ","));
```

A reviewer pointed out the obvious: this is exactly what negative lookbehind was designed for. My hesitation was Safari compatibility — lookbehind landed in Safari 16.4, and the KaTeX user base still has a few Safari stragglers. But checking CaniUse and the project's browserlist, the affected fraction was negligible.

**The lesson:** Sometimes you over-engineer for compatibility that doesn't exist. The manual parser wasn't wrong, but the regex is objectively better — it's declarative, it's testable, it's 1/10th the surface area for bugs.

**The trickier part:** Escaping commas in KaTeX's raw arguments has a hidden trap. `\,` in KaTeX is the thin space command (`\mskip 3mu`), so it gets expanded by the tokenizer before the raw string is assembled. Backslash-escape for commas doesn't survive KaTeX's own parsing pipeline. The real escape path goes through `\\,` (double backslash + comma), which survives as literal backslash-comma in the raw string. This tripped me up in testing.

Unit tests now cover multiple key=value pairs, whitespace handling, and empty-item skipping. All 1290 existing tests still pass.

PR: [KaTeX/KaTeX#4218](https://github.com/KaTeX/KaTeX/pull/4218)
