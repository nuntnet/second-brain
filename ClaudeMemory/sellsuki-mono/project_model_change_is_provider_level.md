---
name: project_model_change_is_provider_level
description: "PO rule 2026-10-12: changing a workspace's answer model requires provider-level permission (sellsuki.provider.llm.update), not company admin; document mode stays a company-admin switch. Enforced in chat-core !140, console !96"
metadata:
  node_type: memory
  type: project
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-11T19:22:48.959Z
---

On 2026-10-12, while the console's model selector was being built, the PO (nuntawit) said: "ใครแก้ไข model ได้ ต้องมี permission ระดับ provider เลยนะ".

**Why:** The model drives cost and answer quality for every message in a workspace. A company admin should not be able to pick it.

**How to apply:**
- Any surface that changes `workspaces.model` must check `sellsuki.provider.llm.update` at `sellsuki.provider:<provider code>` server-side. The permission comes from AI-134, rps migration 0024.
- A hidden button does not count as a check.
- Saving the AI config with the model unchanged stays company-admin.
- chat-core exposes `viewer_can_change_model` on workspace responses.
- Document mode (`/kb-retrieval-settings`) was not restricted. Ask before tightening it.
- If nobody holds the permission on an environment, nobody can change the model in the console. That is fail-closed and intended. Someone with provider authority grants the role.

Related: [[feedback_gemini_25_flash_is_the_kb_answer_model]].
