---
name: rbac-preset-backfill-cards-pat2872-2873
description: "PAT-2872 (Phase 1 auto-reconcile) + PAT-2873 (Phase 2 check from preset) opened 2026-11-17 to end per-permission backfill/migrate; scope must be corrected against AI-252, AI-248, PAT-1527"
metadata:
  node_type: memory
  type: project
  originSessionId: a0c4bf2e-bdf1-481b-bcff-5c37129e1d40
  modified: 2026-11-17T10:31:57.996Z
---

2026-11-17 nunt asked how to stop planning migrate/backfill every time a role or permission is added. Two PAT Task cards opened: **PAT-2872** Phase 1 (reconcile system role presets automatically at deploy, one manifest) and **PAT-2873** Phase 2 (CheckPermission reads role preset from Postgres instead of per-permission-per-user Keto tuples).

**Why:** CCS copies `CompanyOwnerPermissions` into each company's role at creation only; Keto tuples are written per permission per user at assign; system roles are immutable via API, so every new permission needed `rps cmd/backfill_role_permission` or a numbered CCS migration (`_0024`, `_0032` + `make run-migrate-roles-permissions`, see PAT-2850).

**Known gaps in the cards as written (found after opening):**
- A startup reconciler already exists (AI-252, CCS main) but covers only `CompanyRolePresets()` (owner_kind sellsuki.company), not Provider Owner. Phase 1 should EXTEND it, not build a new one. PAT-1527 (open) overlaps: Provider Owner preset + non-idempotent `AdminRefreshProviderRole` (409 instead of additive).
- AI-248 (AI-254/255/257, "shared preset long term") probably overlaps Phase 2; not read yet. Searched only the PAT project, never AI.
- PAT-2747 (audience: company vs provider vs system) should feed the manifest; don't classify by `sellsuki.provider.*` prefix.
- No issue links created between the cards (MCP reverses link direction, cannot delete).
- CheckPermission and CCS role creation code were not read; Keto tuple shape came from docs.

**How to apply:** before touching these cards read AI-248/AI-252 and fix descriptions; Phase 1 must cover CCS migrations too. See [[reference_jira_mcp_search_quirks]].
