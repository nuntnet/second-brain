---
name: rbac-preset-backfill-cards-pat2872-2873
description: PAT-2872/2873 were opened 2026-11-17 then rescoped the same day after reading AI-248/AI-252; AI-252 reconciler is on CCS develop NOT main; phase B (AI-254..258) already covers shared preset
metadata:
  node_type: memory
  type: project
  originSessionId: a0c4bf2e-bdf1-481b-bcff-5c37129e1d40
  modified: 2026-11-17T13:33:53.079Z
---

2026-11-17 nunt asked how to stop planning migrate/backfill whenever a role or permission is added. I opened PAT-2872 (auto-reconcile) and PAT-2873 (check permission from preset), then read epic **AI-248** and found both were largely duplicates. Both cards were rewritten the same day.

**Facts found (verify before relying):**
- **AI-252 (RBAC-A4, Done)** already built the startup reconciler (additive, idempotent, dry-run). Code is on CCS `origin/develop` (`cmd/reconcile_role_presets`) but NOT on `origin/main` (checked 2026-11-17), which matches PAT-2742 (staging lacks it). It covers only company presets (`CompanyRolePresets()`), not Provider Owner (PAT-1527 open) and does not create missing roles.
- **AI-248 phase B** = shared preset: AI-254 (B1 decision, gates B2), AI-255 (CCS stops copying), AI-256 (batch Keto sync), AI-257 (merge thousands of copies per env), AI-258 (retire TUI `migrate_preset_roles`). AI-248 states the migrate-per-env pain is NOT a Keto limit: rps UpdateRole already syncs Keto; the cause is CCS copying presets into per-company system roles that rps refuses to edit.
- PAT-2850 (Done) still wrote CCS migration `_0032` + `make run-migrate-roles-permissions`, so the old path is still in use.

**Current card scope:** PAT-2872 = only what AI-252 lacks (get reconciler onto main/staging/prod, cover Provider Owner + `AdminRefreshProviderRole` additive instead of 409, CI check against per-permission migrations, audience in manifest from PAT-2747). PAT-2873 = a comparison spike feeding AI-254 (resolve-at-check from Postgres role vs shared role vs Keto OPL), no code until AI-254 decides.

**Not done:** no issue links created (MCP reverses direction, no delete); CheckPermission/AssignToRole/UpdateRole not read; no other service-to-Keto grep. See [[feedback_prior_art_sweep_every_jira_project_not_just_target]].
