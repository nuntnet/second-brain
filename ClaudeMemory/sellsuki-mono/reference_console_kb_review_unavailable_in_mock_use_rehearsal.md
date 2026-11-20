---
name: console-kb-review-unavailable-in-mock-use-rehearsal
description: Console KB table mapping/review screens cannot be opened in the mock app (review port returns UNAVAILABLE in demo); POW goes through the DEV-only ?kb-range-review-rehearsal page on the e2e harness dev server
metadata:
  node_type: memory
  type: reference
  originSessionId: ad38d914-b8b9-4919-9b59-07e4ed24311e
  modified: 2026-11-20T15:18:09.172Z
---

`createChatCoreStructuredReviewAdapter(demo)` answers UNAVAILABLE for every call in mock mode on purpose ("no approval in demo"), so `TableMappingReview` / `ReviewPreview` never render in the mock preview and no e2e spec covers them. AI-500 added `apps/admin/src/dev/KbRangeReviewRehearsal.tsx`, a DEV-only page (`?kb-range-review-rehearsal`, gated by `import.meta.env.DEV` in main.tsx, same as `?kb-google-link-rehearsal`) that mounts the real screens with a captured rag-core response.

**How to apply:** for a screenshot, start a full e2e run (`E2E_PORT=<free> npx playwright test --workers=2`) and open `http://localhost:<port>/?kb-range-review-rehearsal` while its dev server is up; set `localStorage admin.locale=th` for Thai. CI serves a prod bundle, so no e2e spec can use the rehearsal. See [[console-new-css-lint-vs-reset-ratchet]].
