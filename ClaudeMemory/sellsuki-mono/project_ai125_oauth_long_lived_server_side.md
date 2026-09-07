---
name: project-ai125-oauth-long-lived-server-side
description: Decision 2026-09-07 — Facebook connect must end with long-lived tokens, which (with AC10's no-token-in-browser rule) means the server-side code exchange, not the browser SDK token handoff
metadata:
  type: project
---

**Decided by the user on 2026-09-07** on AI-125's contradiction (AC10 forbids
any user or page token in browser network traffic, while the code ships one):
the flow **should obtain long-lived tokens**.

Read together with AC10 this settles the contract as the **server-side OAuth
code flow**:
- the browser receives an authorization **code**, never a token;
- chat-core exchanges code → long-lived user token → page access tokens
  (page tokens derived from a long-lived user token do not expire), and stores
  them in Vault.

What the code did at decision time: `facebookSdk.ts` takes a short-lived user
access token from the JS SDK and `chatCoreFacebookConnectionAdapter.ts` posts
it as `short_lived_user_token` — so the browser holds a token, which is what
AC10 prohibits. "Make it long-lived" alone does not fix AC10; only moving the
exchange server-side does, which is why the two are one decision.

Do not weaken AC10 to match the code, and do not treat this as an accidental
persistent-token leak — it is a contract choice that is now made.
