---
name: project-fb-page-dev-mode-gate
description: The AI MVP's real FB page has had exactly one PSID reach the webhook in 10 days — app is in Development mode, so only role-holders' messages arrive
metadata:
  type: project
---

Local AI-chat MVP, verified 2026-08-18.

The **real** page binding is `page_id 268502970215081` → workspace `c11511cc-ecb5-4579-95b0-c5421be87e84`, `token_status: healthy`. (`987654321` is a fake page used only by curl smoke tests — ignore its rows.)

On that real page, `chat.channel_conversation` has had **exactly one peer since 2026-08-08**: PSID `28667647386185932`. Every other row is a `smoke_*` identity I injected.

That is the signature of the **Facebook app being in Development mode**: only users with a role on the app (Admin/Developer/Tester) trigger webhook delivery. Everyone else's message lands in the page inbox and never reaches the webhook.

The decisive evidence is the ABSENCE of a dropped-message trace: messaging-backend writes a `channel_conversation` row on receipt, *before* forwarding to chat-core. No row + no error = Facebook never delivered, so it is not a bug on our side.

To let another person test: add them as **Tester** in App Dashboard → Roles (they must accept the invite), or publish the app via App Review for `pages_messaging`.

The pipeline itself works end to end — a real inbound message got a `tier2` / `gpt-4o-mini` reply in 3 seconds, with `case_id` attributed and continuous across days.

Related: [[project-ai-mvp-integration]]
