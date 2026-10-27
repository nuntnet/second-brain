---
name: project_ai_chat_staging_permission_state_2026_10_27
description: "2026-10-27 audit: why invitees with Company Owner + AI chat admin cannot open AI chat console menus on staging. Company Owner presets come in 3 generations, no preset holds usage/billing, the switcher lists workspaces the user cannot open. Includes how to check"
metadata:
  node_type: memory
  type: project
  originSessionId: 2b0f396b-5941-4a79-8e15-d9e0597e4520
  modified: 2026-10-27T11:30:06.546Z
---

Audit on 2026-10-27 (the PO's question: "I invited someone with Company Owner and every AI chat admin role, and many menus are still blocked"). Facts are from staging rps + Keto + chat-core DB and from `origin/main` code.

**The only gate is chat-core.** The console calls only chat-core (`VITE_API_URL`). Every route sits behind `authctx` with operator or company_admin. In kit v0.8.0 `authctx/roleservice.go`, the company fallback checks **only `sellsuki.chat.workspace.manage`** at `sellsuki.company:<id>`, even for operator routes. A company role holding only `.read` therefore opens nothing.
Usage page: `chat_workspace.usage.read`, and `chat_workspace.billing.read` for the cost part (`use_case/company_quota.go`). Model change: `sellsuki.provider.llm.update` at the provider level, by design ([[project_model_change_is_provider_level]]).

**Company Owner preset on staging, 3 generations** (rps `roles`, `name='Company Owner'`):
- gen1 = no chat code (3,178 roles), e.g. f2bf928c `605106`
- gen2 = legacy `chat_workspace.company_admin` only (498), created around 10-05. chat-core never checks this code, so the result is 403.
- gen3 = `sellsuki.chat.workspace.read/manage` (1,266), created on or after 10-12.
- No generation holds usage/billing. Nor does `AI Chat Company Admin` (1,266 roles, 0 with usage). The only role that has them is the hand-made `Chat Workspace Admin (staging test)` 861542 on f2bf928c.
- CCS main has no reconciler, and Owner is a system role, so UpdateRole refuses it. The tool that can fix it is rps `/app/backfill-role-permission --permission <code> --role-name <name>`: dry-run by default, writes Postgres + Keto. See [[project-ccs-role-presets-apply-only-at-creation]].
- rps 0024 wrote usage/billing into Postgres only, never Keto (OC-4736 ledger).

**Switcher shows workspaces the user cannot open.** `ListAccessibleWorkspaces` labels any role at `sellsuki.company` as `company_admin` (Store Admin included). Example: 5a8905b8 sees test1/test2 (6e0238af/b0e24cb1, gen2 owner) but has manage=n there. Opening them gives 403 on every page.

**Also seen on 10-27:**
- Staging Keto in ns `share` went down from about 16:20 to 16:40 (draining nodes, insufficient cpu). rps readiness 500. chat-core logged `keto tuple operation failed` for both test users. Loki returned `empty ring` at the same time.
- No rps invitation since 2026-10-16. No AI chat role granted in the 7 days before.
- chat-core staging config gaps that look like permission problems but are not: `DASHBOARD_API_BASE_URL` (dashboard 503), `CONSENT_SERVICE_*`, `GOOGLE_SHEETS_SERVICE_ACCOUNT_JSON`, `BOLA_BASE_URL`.

**Backfill dry-run, 2026-10-27 18:01 (rps image 61fdc892):**
- Company Owner: found 3,990.
  - manage/read: grant 3,202 · already 742 · no-tenant 46
  - usage/billing: grant 3,297 · already 0 · no-tenant 693
- Of the 3,202, about 2,228 are real Patona/poshmedica companies, about 974 are automation companies, and only 4 have a chat workspace.
- AI Chat Company Admin: the command finds **0**, because it selects only `is_system_role=true`. The role is non-system (745 roles, all without tenants), so it has to go through UpdateRole or the CCS preset. Assigning it later writes tuples from the Postgres array (`AssignToRole`).
- The command **fails intermittently** with `failed to list roles: keto tuple operation failed`. rps `ListRoles` calls `CountRoleMembers` → `GetRoleByID` for every role, and the e2e suite hard-deletes test companies all day (CCS `companies` n_tup_del 7k+), so one deleted role aborts the whole listing. Retry, and cross-check with SQL over `permissions` + `tenant_refs`. `Plan()` reads only those two columns, so the numbers match.

**Fix plan:** `docs/ai-chat-permission-fix-plan-2026-10-27.md` (5 waves, everything attached to AI-289/AI-252/AI-216/AI-290). Decisions D-046..D-048 are in the Inbox. The plan was uncommitted when written.

**AI-252 trap:** Jira says Done, but the reconciler lives only on `develop`: rps `ff8a21c` (`ReconcileRolePresets` RPC) and CCS `a28c674`. Branch `codex/ai-252-preset-reconciler` was cut from develop and is 90 commits ahead of main, so it cannot be promoted. To get it onto main, cut a new branch from main and cherry-pick both commits. rps must deploy before CCS.

**How to check (read-only):**
- `kubectl -n sellsuki port-forward pod/<rps pod> 25051:50051` (works even when the pod is not ready), then grpcurl `ListAssignedRoles` / `GetRole{role_id}` / `CheckPermission`. ListRoles needs Keto.
- Postgres in `postgresql-pg18-0` (ns datastore, `-U postgres`):
  - `staging_sellsuki_role_and_permission` (roles, invitations)
  - `staging_ory_keto` (`keto_relation_tuples` joined to `keto_uuid_mappings` to read strings)
  - `chat_core` (workspaces)
