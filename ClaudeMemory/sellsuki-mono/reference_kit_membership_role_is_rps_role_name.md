---
name: reference_kit_membership_role_is_rps_role_name
description: "ai-platform-kit-go membership.Member.Role for a company-tier grant is the rps role NAME (\"AI Chat Company Admin\"), never the constant company_admin; a direct workspace grant beside it lives only in DirectWorkspaceRole — build test fixtures with membership.MemberFromRoleTenants"
metadata:
  node_type: memory
  type: reference
  originSessionId: 5b6a49ad-077a-478c-b711-eefdd6aaf899
  modified: 2026-11-02T11:51:46.667Z
---

Found 2026-11-02 (AI-367, verifier FAIL on AC4): a "role" alert recipient resolved nobody for `company_admin` and missed a workspace operator, while unit tests were green.

Kit v0.8.0 `membership/mapping.go` `MemberFromRoleTenants`:
- company-tier grant → `Role = <rps role name>` (`rt.GetRole().GetName()`, e.g. "AI Chat Company Admin", "Company Owner (local dev)"), `ManagedBy = ccs`, covers every workspace of the company;
- direct workspace grant held together with a company role → only `DirectWorkspaceRole = operator` + `DirectWorkspaceIDs` (Role stays the company name);
- workspace-only grant → `Role = operator`, `ManagedBy = workspace`, `WorkspaceIDs`.
- `CoversWorkspace` is true only for `ManagedBy == ccs` or a listed workspace — a fixture with empty ManagedBy covers nothing.

**How to apply:** "who holds role X on workspace W" = `(Role==X && CoversWorkspace(W)) || (DirectWorkspaceRole==X && HasDirectWorkspaceGrant(W))` (chat-core `team.HoldsRole`). Name roles as `GET /members` reports them (`role` / `workspace_role`), not by the kit constants. In tests, build members with `membership.MemberFromRoleTenants`, never by hand.

Related: [[feedback_tests_derive_from_the_card_not_the_code]]
