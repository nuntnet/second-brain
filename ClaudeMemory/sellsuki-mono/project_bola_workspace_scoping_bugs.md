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

**FIX STATUS (2026-07-13): COMPLETE — 65/65 fixed, committed, build+vet green, 0 test failures.** 7 backend commits (17a55f3 batch-1 → ba3dd37) on `fix/bola-workspace-scoping-cross-tenant` + 1 frontend commit on `fix/bola-frontend-workspace-scoping`; monorepo refs bumped. NOT pushed yet (no PR/MR opened). Reusable pattern for future: `GetLineOAScoped(ctx,ws,id)` for line_oa_id handlers; fetch-then-compare for by-id. Follow-up commit 3e759a1 also fixed 2 same-class gaps outside the 65: auto_reply create OA-ownership + rich_menu_assignment apply/evaluate admin routes (Apply pushes rich menu via victim OA token). All pushed to remote; NO MR merged yet.

**5-agent security review (commit cc7bdbb) found ~17 MORE gaps the audit missed — biggest class = `Create*` handlers trusting client line_oa_id/workspace_id.** Fixed: lon sendNotification (LONSubscriberID bypass — req.LineOAID check was decoy → added WorkspaceID param + subscriber-ws check), rich_menu/registration_form/broadcast(×3)/quick_reply/webhook_setting/analytics createEventConfig creates (validate OA ownership; createForm+createEventConfig also stopped trusting req.WorkspaceID), auto_reply log endpoints (no auth/scope), pnp getLogStats+greeting-export trio, auto_push update TargetSegmentID + create/update FlexMessageID ownership, lon consent/revoke callbacks. **LESSON: audit read/update/delete + list well but UNDER-covered Create* foreign-ref validation + sibling twin handlers (getLogStats vs listLogs, greeting-export vs export).** Remaining KNOWN follow-ups (not fixed): 403→404 existence-oracle consistency (follower/registration/auto_reply/quick_reply return ErrForbidden/ErrPermissionDenied=403 vs segment/broadcast/line_oa 404 — MED info leak, needs test churn); pre-existing pnp_template saveAsTemplate SourceID + rich_menu_assignment Apply FollowerID-by-UUID (low). CreateSegment foreign-OA verified SAFE (query ANDs workspace_id → 0 rows).

**(historical) mid-run checkpoint was 48/65 committed.** Branch `fix/bola-workspace-scoping-cross-tenant` (bola-backend) + `fix/bola-frontend-workspace-scoping` (bola-frontend), monorepo refs bumped. Also fixed pre-existing bug: registered `model.ErrPermissionDenied`→403 in helper/errors.go (was 500).
- DONE (committed, build green, tests pass): batch-1 = customer/follower, admin/workspace, line_oa/rich_menu/assignment, registration_form/liff, broadcast (25 incl all 6 critical). batch-2 = segment(6), auto_reply+quick_reply(7), auto_push_message(7), frontend(3).
- REMAINING 17 (all HIGH/MED, same pattern): **lon**(5: subscriber get/revoke by id, sendNotification+list/stats/logs trust body/query line_oa_id, lon_job create), **pnp**(8: send-by-phone, async CSV export, delivery-log list/stats/export + on-greeting, template read/update/delete+list, export-job poll), **contacts-import**(2: use_case/contacts_import.go drops workspaceID before repo), **contact_api_log middleware**(1 insert-missing-ws), **analytics**(1 route trusts foreign ws/line_oa).
- NOT paused by choice: the 6 batch-2 parallel bola-developer subagents died mid-run on **org monthly spend limit**; remaining 3 domains done by hand. GetLineOAScoped(ctx,ws,id) (from batch-1, line_oa.go) is the reusable OA-ownership helper. Test-fixup gotcha: fetch-then-compare fixtures need WorkspaceID set to match the ws arg; ListDeliveryLogs/Delete tests need a GetXByID mock added since the guard now fetches first.
