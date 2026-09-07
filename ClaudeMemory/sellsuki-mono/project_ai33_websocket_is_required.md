---
name: project-ai33-websocket-is-required
description: Decision 2026-09-07 — the admin console's polling transport is rejected; WebSocket-primary is the required pilot transport for the live inbox
metadata:
  type: project
---

**Decided by the user on 2026-09-07:** the admin console must move to
WebSocket. The polling deviation is not approved.

Context: the chat-core WebSocket fan-out (AI-33) works server-side, but
`frontend/ai-chat-admin-frontend`'s `chatCoreConversationAdapter.ts`
deliberately polls instead of subscribing. The completeness audit flagged this
as a transport deviation needing either approval or implementation, and the
answer is implementation.

Consequence for acceptance: a polling implementation may never be recorded as
satisfying a WebSocket AC, and the pilot evidence has to cover cross-replica
updates, reconnect, and event ordering — the things polling silently papers
over. See [[project-ai119-push-deferred]] for the separate push-notification
question (deferred), which is not this.
