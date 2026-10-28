---
name: reference_chat_permission_presets_live_in_ccs_not_rps
description: "The Company Owner / AI Chat Company Admin preset lists are CCS src/use_case/model/permission_group.go, not rps; rps only holds the catalog (migrations) and the backfill-role-permission tool. What chat-core checks per console area."
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ae058e-5385-445d-8aed-f8888c50db6a
  modified: 2026-10-28T16:55:49.532Z
---

Briefs keep saying "fix the rps preset generator". There is none for company roles: CCS writes the roles at company
creation from `CompanyOwnerPermissions` / `AIChatCompanyAdminPermissions` (`permission_group.go`, via `CompanyRolePresets()`).
rps owns the permission catalog (cmd/migration) and `cmd/backfill_role_permission` for existing roles.

Codes chat-core actually checks (read 2026-10-28, kit v0.8.0, AI-289):
- every workspace route: `sellsuki.chat.workspace.read` (operator) / `.manage` at workspace tier; company fallback checks
  **only `.manage`** (`authctx/roleservice.go:159`), even for operator routes
- usage page `chat_workspace.usage.read`, cost `chat_workspace.billing.read` (provider only, D-047), model `sellsuki.provider.llm.update`
- CCS's contract test `permission_chat_workspace_test.go` (from AI-289) lists these with file:line; update it when chat-core adds a check.

backfill-role-permission selects system roles with tenant refs only; AI Chat Company Admin is non-system and mostly
unassigned, so it needs `--include-non-system --grant-unassigned` (AI-289, rps !185 — check it merged before relying on it).
Related: [[project_ai_chat_staging_permission_state_2026_10_27]], [[project_ccs_role_presets_apply_only_at_creation]].
