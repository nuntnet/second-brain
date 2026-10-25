---
name: user-has-vault-and-prod-access-agent-hands-over-commands
description: "nunt has Vault (staging + production) and prod cluster access; for Vault writes / prod tag+deploy the agent should hand over exact commands, not ask \"who has permission\""
metadata:
  node_type: memory
  type: user
  originSessionId: 40764eef-f497-4502-a61b-b77506a8500e
  modified: 2026-10-25T13:07:30.558Z
---

nunt can write to Vault on staging and production (WireGuard `sellsuki-production`) and can tag/deploy prod. Said 2026-10-25 ("ผมมีสิทธิ์ คุณแจ้งมาให้ผมทำ ทำใน terminal หรือ ui ก็ได้").

**Why:** the agent cannot do these itself: classifier blocks prod tag/deploy and Vault writes, and the agent's `tsh` session to the production cluster is often expired (kubectl fails at `tsh` exit 1). Asking "who has the permission?" wastes a round trip.

**How to apply:** when a card needs a Vault key, tag or prod deploy job, give the exact copy-paste commands (value generation included, never echo secrets) and say what to report back; then continue with the parts the agent can do (Jira, MR, pipeline watching). For prod reads (Kafka topics, secrets) ask them to `tsh login` first or run the check themselves. See [[reference_staging_verifier_preflight_tsh_and_pane_login]].
