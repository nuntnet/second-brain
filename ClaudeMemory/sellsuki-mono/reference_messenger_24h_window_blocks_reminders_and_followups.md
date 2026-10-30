---
name: reference-messenger-24h-window-blocks-reminders-and-followups
description: messaging-backend refuses FB sends >24h after the last inbound (outside_24h_window); Meta forbids sales follow-ups outside the window under any tag
metadata:
  node_type: memory
  type: reference
  originSessionId: bb313d1b-3b0c-4ba7-be1b-6115564ef247
  modified: 2026-10-30T16:18:56.663Z
---

- messaging-backend `src/chat/use_case/fb_send_use_case.go:169-173` (origin/main, checked 2026-10-30) returns `SendOutcomeBlocked` with reason `outside_24h_window` when `LastInboundMessageAt` is nil or older than the window (AI-27). A queued retry is abandoned when the window closes. No message tags are sent: `messaging_type` is RESPONSE only (`fb_graph_api_client/http.go:356-373`).
- So any customer-facing sweep that sends days later (lead reminder after 3 silent days, follow-up) cannot reach Messenger customers. It is blocked, not a policy breach.
- Meta policy (policy-overview page, read 2026-10-30): the window is 24h, or up to 7 days after a Click-to-Messenger ad. Tags are not for promotional use; HUMAN_AGENT gives a separate 7-day window for manual human replies. Ways to reach a customer outside the window: Sponsored Messages, One-time Notification (needs the user's opt-in, allows one message), Marketing Messages API (needs App Review; recurring notifications were retired 2026-02-10).
- LINE has no reply window: push messages are allowed but use the OA's message quota.

Related: [[feedback-never-cut-ai-reply-gates-observe-only]]
