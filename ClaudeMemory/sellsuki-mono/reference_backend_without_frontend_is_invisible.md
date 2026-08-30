---
name: backend-without-frontend-is-invisible
description: "GAP markers in this monorepo only ever record 'port with no backend' — the reverse (shipped backend nothing can reach) has no marker and stays invisible for months"
metadata:
  type: reference
---

Found 2026-08-28..30 sweeping AI-9 (E8 Admin Panel). Several chat-core surfaces
were **complete** — routes, tests, OpenAPI, audit — and had **zero frontend**,
so they were unconfigurable in practice. Nothing recorded that:

- `tier1-intents` / `tier1-config` / `tier1-metrics` / `tier1-unmet-queries` (AI-106)
- `anti-abuse-config` / `-metrics` / `-blocklist` (AI-106)
- `hot-lead-notify-config` (AI-103) — the port's GAP note blamed a missing
  BOLA M2M contract, which is real but was never the blocker
- `chat_sessions.rolling_summary` + `GetMostRecentClosedByChannelIdentity`
  (AI-175) — already fed to the model on every reply, never readable

**Why it hides:** every GAP marker convention in this codebase is written on the
*frontend* port ("no backend exists for this yet"). There is no convention for
the other direction, and a backend with no caller produces no error, no failing
test, and no empty screen — there is simply no screen.

**How to check:** diff the route inventory against the FE port registry, not
card status.

```bash
grep -rhno 'Group("[^"]*"\|\.\(Get\|Post\|Put\|Patch\|Delete\)("[^"]*"' \
  backend/sellsuki-chat-core/src/interface/fiber_server/route/*/route.go
grep -n "Port;" frontend/ai-chat-admin-frontend/apps/admin/src/lib/ports.tsx
```

Related: [[flag-without-enforcement]] is the mirror image (UI control the
backend cannot enforce). Both are "the two halves disagree and nothing fails".
Also [[ai-board-in-review-means-merged-unverified]].
