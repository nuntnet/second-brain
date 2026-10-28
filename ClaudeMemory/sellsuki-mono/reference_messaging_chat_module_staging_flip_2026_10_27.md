---
name: reference_messaging_chat_module_staging_flip_2026_10_27
description: "2026-10-27: messaging-backend chat module on staging was switched OFF by !80 (nithiphat 09:59, deploy 10:12, FB routes 404 all day) and back ON by !82 (16:06 → main dddcdbb, deploy 275941 16:46 after 275924 died on aws-cni IP exhaustion). develop never got !80, so no main→develop MR was needed. Six chat keys live in Vault KV kubernetes-secret/sellsuki/sellsuki-messaging-common-secret; VAULT_TOKEN expires ~2026-11-23. Ledger section + M1-M8 morning checklist in docs/cards/AI-216.md on branch docs/AI-216-reopen-20261027"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4683d2cf-08fe-4531-bc03-8b12af31b5cb
  modified: 2026-10-28T06:24:30.143Z
---

What happened (all from GitLab API project 568, not recalled):
- !80 `077ebd3` disabled `CHAT_MODULE_ENABLED` and removed the six secret refs "so OTP/SMS/email can start without chat secrets". The root cause of the morning boot failure was never written down (Decision Inbox D-049).
- !81 set production `CHAT_MODULE_ENABLED: "false"` explicitly; production `v1.1.0` = `dddcdbb9` deployed 20:18 ICT with the flag off.
- !82 `9c52b56` restored staging byte-for-byte. First deploy 275924 failed with `aws-cni … failed to assign an IP address to container` (node IP pool, not the module); retry 275941 passed Helm `--atomic --wait` (rev 37).
- nunt's other session opened Draft !83 at 16:07 for the same revert; closed as duplicate.

Boot facts worth keeping: `chatUC.HealthCheck` (helper.go:540-549) pings Postgres, Vault `/sys/health` (no token), Kafka `WriteMessages(ctx)`, Redis `PING`, chat-core `/system/liveness`, and `slog.Fatal`s before Fiber starts — so a passing atomic deploy proves every secretKeyRef resolved and those five answered. It does NOT prove `VAULT_TOKEN` is valid (403 surfaces per-webhook as 503 `fb_app_secret_unresolvable` after AI-326) nor that the two service-token pairs still match chat-core.

Verified 2026-10-28 13:20 ICT on staging (read-only kubectl): image dddcdbb9, six keys non-empty, clean log, in-pod lookup-self 200 with `expire_time 2026-11-23T13:11:22Z` read from Vault (policies default + messaging-chat-staging, renewable), chat-core liveness 200, four route probes equal to the 09-21 baseline. Still open then: the real FB round-trip (M7, nunt on Messenger) and D-049 (why !80 happened — needs `vault kv metadata get`).

**How to apply:** before touching messaging values for the chat module, `glab api projects/568/merge_requests?search=AI-216&state=all` first — the flip has now been done three times by different people. For the morning check and the Vault verify/renew block see `docs/cards/AI-216.md` (2026-10-27 section). Related: [[reference_messaging_backend_staging_vault_token_403_makes_fb_webhook_500]], [[reference_ai_agent_staging_vault_token_expiry_crash_on_restart]], [[reference_night_window_is_staging_th_only_production_runners_run]].
