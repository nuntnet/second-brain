---
name: reference_connector_403_is_often_chat_core_token_mismatch
description: "A 403 from knowledge-connector (Drive OAuth, sources) can be chat-core's 401 to its workspace lookup in disguise: connector CHAT_CORE_SERVICE_TOKEN must equal chat-core SERVICE_TOKEN; respondError maps unknown errors to 403 and logs nothing. How to prove it in-cluster without printing tokens"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-11T11:33:19.061Z
---

**Seen on staging 2026-10-11 (AI-49):** the console's Connect Google Drive got 403 from
chat-core `/v1/workspaces/:ws/knowledge-google/oauth/begin`. chat-core only relays; the 403 was
the connector's. Its `CanManageDrive` first calls chat-core
`GET /internal/workspaces/:ws/company` with `X-Service-Token: $CHAT_CORE_SERVICE_TOKEN`.
chat-core accepts only its single `SERVICE_TOKEN`; the two secrets differed → chat-core 401 →
`ErrWorkspaceResolverUnavailable` → the connector's `respondError` default branch → **403 "forbidden",
no log line**. Looks exactly like a missing permission; it was not (the user held
`sellsuki.rag.connector.create` in Keto).

**How to apply:**
- Before chasing permissions on a connector 403, run a throwaway pod (postgres:16-alpine has busybox
  wget) with both secret keys as env and compare them in-shell (`[ "$T" = "$C" ]`), and call the
  lookup with the connector's token — 401 means token mismatch.
- Fix direction: copy chat-core's `SERVICE_TOKEN` into the connector's `CHAT_CORE_SERVICE_TOKEN`
  (other services already use chat-core's value). Vault path
  `kubernetes-secret/sellsuki/sellsuki-knowledge-connector-secret`; `scripts/seed-google-drive-secrets.sh
  sync-chat-token` does it without printing. Then force-sync the ExternalSecret and
  `rollout restart deploy/knowledge-connector` (env is read at start).
- Staging chat-core does not log requests, so the browser console + in-cluster probes are the evidence.

Related: [[reference_ai_chat_staging_is_ns_sellsuki_and_how_to_release_there]], [[reference_keto_staging_permission_lookup]].
