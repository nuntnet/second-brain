---
name: reference_permission_code_underscore_rejected_before_query
description: rps PermissionCode.Validate rejected underscores, and GetPermissionByCode validates before it queries — so an underscored code was unreachable no matter what the catalog held, and CCS dropped it silently from the invitation screen
metadata:
  type: reference
---

`sellsuki-role-and-permission-management-backend` validated permission codes with
`^[a-z][a-z0-9]*(\.[a-z][a-z0-9]*)+$` — **no underscores anywhere**. The gRPC
handler calls `code.Validate()` *before* it touches the database
(`role_and_permission_service_server.go` GetPermissionByCode), so an underscored
code answered `InvalidArgument` whether or not the row existed. Seeding it would
not have helped.

Widened 2026-09-11 (MR !122 main / !123 develop) to allow underscores **inside**
a segment but not as the separator: `chat_workspace.provision` valid,
`patona_store_create` still invalid (a dot is still required).

**Why it was invisible for months.** CCS's `GetJoinInvitation`
(`src/use_case/invite.go`) resolves each of a role's codes and `continue`s past
any it cannot resolve — originally with no log at all. A role whose codes are all
unresolvable therefore returns `permission_item: []`, HTTP 200, and the
invitation screen renders an empty card. Every layer reports success. CCS !330
added a WARN listing the dropped codes.

**Codes that were affected:**
- `sellsuki.messaging.config.set_primary` — in the catalog since rps migration
  0018, unreachable through this RPC that whole time.
- `chat_workspace.provision` / `chat_workspace.company_admin` — never seeded at
  all, though CCS's `CompanyOwnerPermissions` preset carries both and
  `ai-platform-kit-go`'s `authctx` enforces the second. Seeded by rps 0023.

**Checks worth repeating:**
- `select code from permission_lists where code ~ '_'` — anything there is a
  candidate for this class of failure on an older deploy.
- A role showing an empty permission list on the invite screen means "codes did
  not resolve" far more often than "role grants nothing".

Related: [[reference_rps_dual_mainline]] (0023 had to clear both mainlines —
main was at 0022, develop at 0021) · [[reference_ambiguous_404_fail_open]] ·
[[project_pointclaim_permission_missing_from_owner_preset]] (same family: a
permission that exists in code but not in the catalog).
