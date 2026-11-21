---
name: reference-vite-stale-proxy-after-checkout-switch
description: "CCS2 \"Something went wrong\" on local = Vite kept an old proxy config after a branch switch; requests skipped Caddy's Kratos→X-User-Id and CCS answered 401. Fix: overmind restart web-provider."
metadata:
  node_type: memory
  type: reference
  originSessionId: 58c6eca6-d6eb-446b-bf6a-a91ba8b63329
  modified: 2026-11-21T08:04:38.724Z
---

2026-11-21: provider.sellsuki.local showed "Something went wrong" for a user who did have permission (direct CCS call with their X-User-Id → 200).

Cause: the running Vite process (started 12:15) still used the pre-80dac1c proxy `/ccs-api → http://localhost:8092`, although vite.config.ts on disk (rewritten 12:17 by a local/stack checkout) points at `https://ccs.sellsuki.local`, the Caddy host whose forward_auth turns the Kratos `sellsuki_session` cookie into X-User-Id. Without it CCS returns 401 → provider store sets `error` (not `denied`) → generic error page.

How to tell: the CCS log shows `Host: localhost:8092` + `X-Forwarded-Host: provider.sellsuki.local` and no `X-User-Kind`. A request through ccs.sellsuki.local with no session never reaches CCS (Kratos 401).

Fix: `overmind restart web-provider -s .overmind-web-provider.sock` (see [[reference-overmind-start-d-does-not-start-one-process-use-restart]]). In CCS2, 403 = AccessDeniedPage; any other failure = "Something went wrong".

CCS1 follow-up (same day): CCS1 called ccs./accounts./localhost:8085/8088 cross-host with a fixed X-User-Id → the desktop built-in browser blocks it (ERR_BLOCKED_BY_CLIENT, "Could not load provider"). Branch `fix/PAT-2735-ccs1-same-host-proxy` (abb9fbe) moves CCS1 to /ccs-api, /config-api, /i18n-api, /ams-api. Dropping the fixed id exposed that i18n.sellsuki.local had no forward_auth → "unauthenticated" 500 → auth-redirect login loop (screen flicker); fixed in monorepo 52e9caf08. Signed-in user needs seed-dev run with their own DEV_USER_ID for system roles.
