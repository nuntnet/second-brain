---
name: reference_procfile_messaging_disables_chat_module
description: Procfile.messaging starts messaging-backend with CHAT_MODULE_ENABLED=false, so /webhook/fb/{app} answers 404 while the port looks perfectly healthy — days of FB webhook debugging were spent against a binary that could not receive a webhook at all
metadata:
  type: reference
---

`Procfile.messaging` (root of the monorepo, untracked) runs:

```
messaging: ... PORT=8100 CHAT_MODULE_ENABLED=false "$W" 8100
```

That flag drops the **entire chat module**, so the process holding port 8100
answers:

- `/webhook/fb/{app_id}` → **404** (Meta's delivery bounces, silently)
- `/v1/fb-apps/*` → 404
- `/metrics`, `/healthz`, the OTP/SMS routes → 200, perfectly healthy-looking

## Why this burned a lot of time (2026-09-17)

Facebook messages "stopped arriving" for a workspace. Every hypothesis chased
was on **Meta's side** — page subscription, Development vs Live mode, app
secret, handover protocol, OAuth flow (see
[[reference_fb_page_must_be_subscribed_to_app]]). Three successive "root causes"
were announced and all three were wrong, because the webhook endpoint was
returning 404 the whole time.

There is more than one overmind session on this machine
(`.overmind-ai-mvp.sock`, `.overmind-messaging.sock`, …). **Whichever one grabs
port 8100 first wins**, and the port being bound tells you nothing about which
Procfile — and therefore which env — the listener came from.

## The check that costs five seconds

Before blaming any third party for "not delivering", prove our own endpoint
exists:

```bash
curl -s -o /dev/null -w '%{http_code}\n' localhost:8100/webhook/fb/<app_id>
```

`400` = route is live (it is rejecting an unsigned request, which is correct).
`404` = the chat module is off; nothing external can possibly work. Also check
which session owns the port: `lsof -nP -iTCP:8100 -sTCP:LISTEN` and read that
session's Procfile before trusting anything else.

Generalised: **a 404 from your own service looks identical to "they never sent
it".** Confirm the bottom layer serves before reasoning about the top one.
