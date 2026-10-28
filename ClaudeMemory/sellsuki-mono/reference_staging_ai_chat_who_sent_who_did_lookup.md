---
name: reference_staging_ai_chat_who_sent_who_did_lookup
description: "Staging AI chat: resolve a Messenger PSID to the sender's name via chat_core.chat_sessions.customer_display_name (db is `chat_core`, NOT staging_-prefixed), and an audit actor uuid to a person via staging_ory_kratos identities.traits — no need to ask the tester"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2b0f396b-5941-4a79-8e15-d9e0597e4520
  modified: 2026-10-28T14:53:36.058Z
---

Learned 2026-10-28. The goal was to confirm that a new PSID belonged to the tester without asking him to check.

- **Sender name for a PSID.** chat-core's database on pg18 (ns `datastore`, pod `postgresql-pg18-0`) is called **`chat_core`**. It is not in `select … like 'staging%'`, so a sweep of the `staging_*` databases misses it. Its URI comes from a secret in `values-staging.yml`, so don't try to read it from the deployment.
  `select customer_display_name, … from chat_sessions where workspace_id='…' and channel_identity_ref like '%<PSID>%'` returned `Udomdej Chuenjit`.
  `messages` (with `sender_type` customer/ai and `delivery_status`) holds the counts and the delivery result. messaging-backend's `chat.channel_conversation` has only `peer` (the PSID) and no name.
- **Person behind an `actor_user_id`.** Run `select traits->'name' from identities where id='…'` in **`staging_ory_kratos`**. The `_2` and `_3` copies don't have it.
- **Unbind is a hard delete.** `chat.fb_page_binding` has no `deleted_at`, so an unbound page simply has no row. Its history (actor, time) exists only in `chat.fb_app_audit_log`.
- **zsh trap.** `P="kubectl … psql"; $P -c …` fails with "command not found" because zsh does not word-split. Use a function: `P(){ kubectl … -- psql -U postgres "$@"; }`.

Related: [[reference_staging_fb_webhook_404_is_page_unbound_silently]], [[reference_staging_fb_app_is_a_test_app]].
