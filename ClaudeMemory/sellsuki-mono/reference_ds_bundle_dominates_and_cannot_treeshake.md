---
name: ds-bundle-dominates-and-cannot-treeshake
description: "@sellsuki-org/sellsuki-components is 94% of ai-chat-admin-frontend's initial chunk and cannot be tree-shaken — single export, no sideEffects field"
metadata:
  type: reference
---

Measured 2026-08-31 on `frontend/ai-chat-admin-frontend` (AI-119, MR !28).

```
initial chunk                                     2,367 kB
  @sellsuki-org/sellsuki-components dist/main-*.js  2,216 kB   (94%)
  everything this app wrote                        ~  147 kB
```

**Why it cannot be tree-shaken.** Its `package.json` declares one export
(`"."` → a single pre-bundled file) and **no `sideEffects` field**, so a
bundler must assume every module in that 2.2 MB has side effects and keep all
of it. Importing the 26 components the app wraps pulls in all 99, plus its
dependencies — tiptap, croppie, date-fns, change-case. The package also ships
a 1.9 MB `TheShibas.png`.

**Lazy-loading the DS is not a workaround.** Custom elements must be
registered before first render, so deferring it moves 2.2 MB to a second
request and delays first paint instead of shrinking anything.

**What DID help:** route-splitting the 18 non-landing pages —
2,548 → 2,367 kB (−7.1%), gzip 653 → 615 kB. Keep `RootRedirect` and
`InboxPage` eager; they are the landing screens and deferring them only adds a
round trip where content exists today.

**What did NOT help, so don't retry it:** declaring `sideEffects: false` on
our own three workspace packages produced a **byte-identical bundle** (same
content hash). Reverted — no measured gain, and a real tail risk of a bundler
dropping a module later.

**The actual fix is upstream:** `sideEffects: false` plus per-component entry
points in the DS. That benefits every consumer, not just this app. Raised in
MR !28 rather than worked around.

Related: [[ds-1-0-beta-gotchas]], [[ssk-components-docs]].
