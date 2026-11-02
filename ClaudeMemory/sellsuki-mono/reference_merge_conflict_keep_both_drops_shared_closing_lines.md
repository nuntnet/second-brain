---
name: reference_merge_conflict_keep_both_drops_shared_closing_lines
description: "Resolving an add/add git conflict by concatenating \"ours\"+\"theirs\" drops the hunk's shared trailing lines (closing braces) — broke the whole console index.css; unit tests never load CSS, only the e2e mock build showed vite's overlay"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9bfbf442-2ea7-482e-8e36-8246c00e89fd
  modified: 2026-11-02T21:02:45.570Z
---

Seen 2026-11-03 (AI-371 merging the AI-370 shell head): both branches appended a CSS block at the same line. git put the common trailing `}` lines OUTSIDE the conflict markers, so a scripted "keep both" left two blocks unclosed. tsc + 2000 unit tests green; every e2e route then failed with postcss "Unclosed block".

**How to apply:**
- When both sides only append at one point, rebuild instead of editing markers: `base[:n] + ours_added + theirs_added + base[n:]` (check each side's `git diff -U0 base side` is a single `@@ -n,0 … @@`).
- After any CSS merge, parse it: `node -e 'require("./node_modules/.pnpm/node_modules/postcss").parse(fs.readFileSync(f,"utf8"))'`.
- In zsh, never write `$REF:path` (`:a` is a path modifier) — use `${REF}:path` or do git calls from Python.
Related: [[reference_ssk_page_title_takes_children_not_title_prop]].
