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
