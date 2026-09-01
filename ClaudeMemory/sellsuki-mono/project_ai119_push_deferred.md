---
name: ai119-push-deferred
description: "Web Push (AI-119 AC8/AC9) deprioritized 2026-08-31 by the user — and it is NOT the live-inbox update, which already works via AI-33 WebSocket"
metadata:
  type: project
---

**Decision (user, 2026-08-31): "มันยังไม่ใช่ scope ที่รีบ"** — push is not
urgent. The bundle/perf third of AI-119 shipped as MR !28; auth-in-standalone
and push are not being pursued for now.

**The confusion worth not repeating.** Push was initially read as "the inbox
updates without a refresh". It is not — those are two different mechanisms:

| | WebSocket (AI-33) | Web Push (AI-119) |
|---|---|---|
| fires when | the app is OPEN | the app is CLOSED / screen locked |
| shows | messages flowing into the list in-page | an OS-level notification outside the app |
| status | **already on main and working** (`route/conversation` `Get("/ws")`, FE `useWorkspaceRealtime.ts`) | no infra anywhere |

So live updating is done; push only extends coverage to when nobody is looking.

**Verified absent** across chat-core, messaging-backend and ai-agent
(`grep -i 'vapid|firebase|fcm|apns|device_token|web-push'` on `origin/main`
→ empty in all three). Building it means: VAPID keypair, per-device
subscription store, sender, and pruning on 404/410.

**Two constraints that shape whether it is even worth it:**

- **iOS gives web push only to a home-screen-installed PWA** (Safari 16.4+) ⇒
  standalone auth (AC2–AC7) must work FIRST; push cannot be done in parallel.
- **A fully-quit desktop browser gets nothing** — no process for the OS to
  wake. Push helps phone-first admins; it does almost nothing for a team
  working at a desktop that gets shut down.

**If it is revived:** owner should be **messaging-backend**, not chat-core.
It already owns per-destination credentials that expire and need reaping
(`fb_page_bindings.token_ref/token_status/token_last_checked_at` plus AI-26's
health check) — a push subscription is the same problem. chat-core already
calls it on every outbound send, so the hop exists.

**Two traps to carry forward:** a subscription is per BROWSER, not per
workspace, so fan-out must resolve "may this device's user see workspace X"
at send time (that is where AC9's no-cross-workspace rule breaks silently);
and the notification body must not carry customer message text — it lands on
a lock screen, same reasoning as the card's own cache/PII rule.

Also blocked regardless: admin has **no per-conversation route**, so a tapped
notification can only land on `/w/:workspaceId`, not the right thread.

Related: [[ds-bundle-dominates-and-cannot-treeshake]], [[e8-remaining-blockers]].
