---
name: chatcore-company-list-derived-not-asked
description: "chat-core's company list must be derived from ListAccessibleWorkspaces, not from CCS ListCompanyThatHaveAccess, which returns empty for workspace-tier operators"
metadata:
  type: project
---

Decided and shipped 2026-08-28 (AI-94, chat-core MR !39, `GET /v1/me/companies`).

CCS has an RPC that looks purpose-built for "which companies may this user
see" — `ListCompanyThatHaveAccess(provider_code, identity)`. **Do not use it
here.** It resolves access via rps `ListAssignedRoles` and keeps only grants
whose kind is `sellsuki.company`.

chat-core **grants `sellsuki.chat_workspace`** and never `sellsuki.company`
(`role_assignment_repository.TenantKindChatWorkspace`, written by
`AssignOperator`). So that RPC answers **an empty list** for exactly the
operators the admin panel serves, while the workspace switcher beside it lists
their workspaces. Nothing errors — the classic silent wrong answer.

The company set is therefore derived from `ListAccessibleWorkspaces`, which
already handles both grant tiers. Consequences worth keeping:
- a company appears **iff** the user can open a workspace in it
- no second authorization path to keep in sync
- cannot drift from the switcher, because it is computed from it
- **no read-by-id endpoint**: CCS's `GetCompanyById` has no caller identity, so
  a by-id proxy is an IDOR guarded only by vigilance. Filter the list instead.

**Invariant that makes a defensive branch nearly dead:** `ListByCompany`
returns every workspace of a company *unfiltered*, so a company-tier grant
makes all of them `company_admin` and every workspace-tier grant inside is
skipped as already-covered. Within one company all roles are therefore
identical — the role-widening branch only fires when `ResolveCompany` and
`ListByCompany` disagree (a row missing from the list read). A test for it must
construct that disagreement **and** make the operator row sort first, since the
list is sorted by name. Two hollow-test rounds were needed to get there; see
[[timing-dependent-concurrency-tests]] for the same lesson in another shape.

**Ref-vs-enum, answered:** `CompanyProfile` carries **no** package tier, plan,
or contract note. AI-94's form asks for a `package_tier` enum but CCS's company
record has no such field, and AI-94's own Out of Scope puts plan definition in
management-backend/AI-6. Storing the enum in chat-core would make it a second
owner of "which plan is this company on" against
[[plan-capability-quota-anchor]], and the owners would drift silently because
both compile. A plan travels as a **reference** resolved by its owner.

`createCompany` stays unavailable on purpose: CCS's `CreateCompany` needs an
owner identity + provider code, and making chat-core a writer of the company
registry is a product decision. CCS has its own management frontend.

Related: [[ccs-go-module-path-broken]], [[chatcore-is-the-admin-bff]],
[[rps-identity-kind-must-be-prefixed]]
