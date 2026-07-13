---
name: project_bola_workspace_scoping_bugs
description: BOLA multi-tenant split left 65 confirmed cross-tenant scoping bugs — systemic missing workspace_id check on by-ID + line_oa_id handlers
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
---

BOLA SaaS multi-tenant (split by workspace) conversion left **65 confirmed workspace-scoping bugs** (audit 2026-07-13, 13-agent fan-out + adversarial verify). Full checklist: `backend/bola-backend/docs/workspace-scoping-audit-TODO.md` (6 critical / 50 high / 7 medium / 2 low).

**One systemic bug repeated everywhere:** workspace boundary was applied to list + create-from-session paths (which source `caller.WorkspaceID`), but NOT to:
1. single-record `GET/PUT/DELETE /:id` handlers — fetch/mutate by UUID with no workspace_id check → cross-tenant read/edit/delete by ID guessing
2. list/action handlers scoped by client-supplied `?line_oa_id=` — trusted, never checked vs caller.WorkspaceID
3. create/link accepting foreign `line_oa_id`/`webhook_setting_id`/target IDs (cross-ws-reference)
4. async export/import jobs — stored under caller's ws but process a foreign line_oa_id, so the later ownership check passes on other-tenant data

**Correct pattern (reference):** `src/repository/customer_repository/postgres_gorm.go` + phone_contact fetch-then-compare in `follower.go`. Repo method takes `workspaceID` arg → `.Where("workspace_id = ?", workspaceID)`; route threads `caller.WorkspaceID`. Fix = add filter (prefer, → 404 not 403 oracle) or fetch-then-compare `record.WorkspaceID != workspaceID`. Add 2-workspace leakage test per fix (`workspace_scope_test.go`).

**Why:** the split predates this audit; leak signal is invisible because unscoped queries compile and return data. **How to apply:** when touching any BOLA by-ID or line_oa_id handler, verify workspace scoping first; hardest-hit clusters = auto-push/auto-reply/quick-reply (14), richmenu/liff/registration (11), pnp (8). Relates to [[project_bola_auth_mode_deployment]].

**FIX PROGRESS (as of 2026-07-13): 48/65 committed.** Branch `fix/bola-workspace-scoping-cross-tenant` (bola-backend) + `fix/bola-frontend-workspace-scoping` (bola-frontend), monorepo refs bumped. Also fixed pre-existing bug: registered `model.ErrPermissionDenied`→403 in helper/errors.go (was 500).
- DONE (committed, build green, tests pass): batch-1 = customer/follower, admin/workspace, line_oa/rich_menu/assignment, registration_form/liff, broadcast (25 incl all 6 critical). batch-2 = segment(6), auto_reply+quick_reply(7), auto_push_message(7), frontend(3).
- REMAINING 17 (all HIGH/MED, same pattern): **lon**(5: subscriber get/revoke by id, sendNotification+list/stats/logs trust body/query line_oa_id, lon_job create), **pnp**(8: send-by-phone, async CSV export, delivery-log list/stats/export + on-greeting, template read/update/delete+list, export-job poll), **contacts-import**(2: use_case/contacts_import.go drops workspaceID before repo), **contact_api_log middleware**(1 insert-missing-ws), **analytics**(1 route trusts foreign ws/line_oa).
- NOT paused by choice: the 6 batch-2 parallel bola-developer subagents died mid-run on **org monthly spend limit**; remaining 3 domains done by hand. GetLineOAScoped(ctx,ws,id) (from batch-1, line_oa.go) is the reusable OA-ownership helper. Test-fixup gotcha: fetch-then-compare fixtures need WorkspaceID set to match the ws arg; ListDeliveryLogs/Delete tests need a GetXByID mock added since the guard now fetches first.
