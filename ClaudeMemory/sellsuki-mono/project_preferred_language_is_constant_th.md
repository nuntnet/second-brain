---
name: project-preferred-language-is-constant-th
description: chat_session.preferred_language is hardcoded "th" for every session — no detection, no write path — so any AC leaning on it is leaning on a constant
metadata:
  type: project
---

In `backend/sellsuki-chat-core`, a session's `preferred_language` is **not the
customer's language**. It is a default nobody ever sets:

- `chat_reply.go` resolves the session with `GetOrCreateOpenSession(ctx,
  workspaceID, channelIdentityRef, "")` — an empty language, hardcoded.
- The use case and repository both fill an empty value with
  `chat_session.DefaultPreferredLanguage` = `"th"`.
- No route writes the field (the conversation DTO only reads it), and nothing
  detects language from the customer's text.

So every session on every channel is `"th"`. Any acceptance criterion phrased
as "respect the session's preferred_language" therefore means "always answer in
Thai", including to a customer writing English — which is almost certainly not
what such a card intended (AI-135 was written this way).

**Open decision, raised on AI-135 on 2026-09-07 and not yet answered:** (a)
detect the language from the first message and persist it, (b) let an admin set
a per-workspace default and pass it into `GetOrCreateOpenSession`, or (c) accept
that the pilot is Thai-only and document the field as a constant.

Until then, AI-135's fix in `backend/sellsuki-ai-agent` (merged) instructs the
model to answer in the configured language *but to follow the customer's own
language when it is clearly different* — deliberately a superset of the AC, so
the reported bug (empty question → Thai) is fixed without inventing a
Thai-only policy. Related: [[project-ai-chat-platform-plan]].
