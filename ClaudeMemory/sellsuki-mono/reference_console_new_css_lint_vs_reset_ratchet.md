---
name: console-new-css-lint-vs-reset-ratchet
description: "ai-chat-admin-frontend new CSS can use neither spacing family — ESLint sellsuki-ds rejects --ssk-spacing-N, consoleReset.guard ratchet bans --spacing-*; also bans SskAlert type=info and i18n strings >120 chars"
metadata:
  node_type: memory
  type: reference
  originSessionId: ad38d914-b8b9-4919-9b59-07e4ed24311e
  modified: 2026-11-20T15:18:04.246Z
---

In `frontend/ai-chat-admin-frontend` (since AI-402, 2026-11-20) two guards contradict for any NEW stylesheet:

- `eslint` `sellsuki-ds/css-size-from-token` errors on `var(--ssk-spacing-N)` (not a real DS token; old files are on an allow-list in `eslint.config.js`).
- `src/test/consoleReset.guard.test.ts` (unit suite) ceilings ban `var(--spacing-*)` (count must not go up).

So a new screen/dev page uses Tailwind layout classes (`flex flex-col gap-lg p-lg`) and no CSS file. The same ratchet also fails on a new `SskAlert type="info"` and on any th/en.json string longer than 120 characters — keep new copy short.

**How to apply:** run the guard test (`npx vitest run src/test/consoleReset.guard.test.ts`) with lint before calling UI work done. Related: [[admin-fe-testid-shifts-axe-baseline-and-turbo-loose-env]].
