---
name: ai150-members-read-only
description: "AI-150 members page is read-only by design, not unfinished — every grant the platform can express is company-tier and CCS owns it"
metadata:
  type: project
---

Built 2026-08-30 (chat-core `794b084`, frontend `cf56b33`, branch
`feat/AI-129-playbook-history` / `feat/AI-96-kb-entries`).

`GET /v1/workspaces/:workspace_id/members` — **workspace-scoped, not
company-scoped**, though the data is a company's. `authctx` enforces against a
workspace in the path and there is **no company-scoped guard** in chat-core or
in ai-platform-kit-go. A second guard would be a second authorization path for
one page. Side effect: "another company's id → 403, not empty list" is true
without writing anything for it.

**Read-only is the answer, not a first slice.** Company-tier grants are CCS's
to administer; workspace-tier grants cannot exist because rps rejects
`sellsuki.chat_workspace` (entity!43, tracked as AI-183). So no grant this
console can legitimately write. A test fails if a write route is registered.

**AI-182's "3 remaining items" were all still unbuilt** when checked — chat-core
did not import the kit's `membership` package at all. The kit itself was
complete (`Member`, `ManagedBy`, `MemberFromRoleTenants`, `ListByCompany`,
`CoversWorkspace`).

**The gap the card missed:** `membership.Member` carries only `UserID` by
design, and chat-core has no users table and no Kratos admin URL — so the page
would have rendered a raw id as the person's name. Solved by vendoring CCS
`UserService.GetUserProfile` as a trimmed proto into
`company_directory_repository` (same reason as company_service: CCS's go.mod
declares a module path that does not exist). Only first/last/email crosses the
wire.

Still blocked, matching the card: grant/revoke company_admin, no-delete-last,
Rule 12 (CCS/rps own those writes; kit has no `GrantCompanyRole`), and the
5-state invitation list (kit has no `ListInvitations`/`RevokeInvitation` — bump
the kit, never patch at chat-core or FE).

Related: [[entity-lib-tenant-kinds]], [[rps-identity-kind-must-be-prefixed]],
[[backend-without-frontend-is-invisible]].
