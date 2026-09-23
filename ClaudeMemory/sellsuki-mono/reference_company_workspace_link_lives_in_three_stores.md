---
name: reference_company_workspace_link_lives_in_three_stores
description: "The company↔BOLA-workspace relationship is written into THREE databases by TWO creation paths with nothing reconciling them — CCS's own bola_workspaces registry (3 rows), bola.workspaces.company_id (4 of 7,514) and crm.oc2plus_bola_bindings (6,857); the CCS screen reads only the first"
metadata:
  node_type: memory
  type: reference
---

Established 2026-09-23 while chasing "หน้า CCS → BOLA Workspaces ว่าง".

| store | columns | written by |
|---|---|---|
| `ccs.bola_workspaces` | company_id, bola_workspace_id, slug, name, plan_id, status | CCS, **only** when creation goes through CCS |
| `bola.workspaces` | company_id, company_name | BOLA, when the create/update carries them |
| `crm.oc2plus_bola_bindings` | company_id, bola_workspace_id, slug | OC2Plus CRM at bind time |

Staging counts: **3 / 7,514 / 6,857** — no two agree.

🔴 **`slug` appears in all three but is NOT one fact — do not reconcile it.**
`crm.oc2plus_bola_bindings.slug` is the MEMBER-APP link
(`member.staging.oc2.plus/<slug>/register`, renamed from the "ชื่อในลิงก์" field,
OC-4560). `bola.workspaces.slug` is BOLA's own workspace identifier, used to build
its invite URLs (`accept-invite?token=…&workspace=<slug>`,
`bola-backend/src/use_case/admin.go:234` and `:1610`). They start equal only
because CRM passes `company.Code` as both at create time. Renaming the member-app
link correctly leaves BOLA and CCS untouched — verified on `Supermarket`
2026-09-23: CRM `supermarket` @08:35, the other two still `dapo83bshnmbdpn3qspg`
@08:26. **Syncing them would break every BOLA invite URL.** A real user read this
as a bug once already; the obvious "fix" is the destructive one.

**The CCS screen does not call BOLA.** `ListBolaWorkspaces` → `ListByCompanyID`
reads CCS's own table. And BOLA could not answer anyway: its
`GET /v1/system/workspaces` takes only `page` / `page_size`, there is no
`company_id` filter.

So fixing `bola.workspaces.company_id` (OC-4598) does **not** make that screen
show anything — it fixes `MembershipManagedBy()` (`ccs` vs `local`, which decides
whether BOLA's Team Members page allows invite/remove) and the PERSONAL
WORKSPACES grouping. Two different problems that look identical from the screen.
OC-4598's AC-02 claimed the screen would light up; that was wrong and has been
corrected on the card.

**Why the CCS table is not itself the bug:** it is an entitlement record, it is
what `ListBolaWorkspaces` authorizes against before reading (BOLA-310 — an owner
sees only their own), and the screen should not go blank when BOLA is down.
Authorizing from data you must fetch from the service you are gating access to
is backwards. The defect is **two writers and no owner**, not the existence of a
local table.

Mechanism: `CREATE_BOLA_WORKSPACE_VIA_CCS` has `envDefault:"false"` and appears
in no `values-*.yml`, so every environment creates straight against BOLA and
CCS's registry never learns. BOLA-317 built the through-CCS path precisely so the
registry is written in the same call; it is switched off everywhere.

Carded as **OC-4599** with three options (turn the flag on / have CRM register
via `POST /v1/companies/{id}/bola-workspaces/bind` after creating / backfill) and
a proposal that CCS be the writer and the other two derive. That is an ownership
change across three services — a product decision, per
`.claude/rules/design-doc-authority.md`, not a refactor.

Same shape as [[reference_bola_follower_metadata_two_stores]] and the rule in
`.claude/rules/oc2plus-service-boundary.md`.

## ทางซ่อมที่พิสูจน์แล้ว (2026-09-23, staging)

`POST {CCS}/v1/companies/{companyID}/bola-workspaces/bind` with
`{bola_workspace_id, name, slug}` and the owner's `X-User-Id` / `X-User-Kind`
registers an EXISTING BOLA workspace into CCS's registry and the CCS screen
lights up immediately. It returns **200, not 201**, and touches nothing in BOLA —
both deliberate, per the route's own comments ("must NOT create anything in
BOLA"). Verified end to end on `Supermarket`.

`GET /v1/bola-workspaces/registry-drift` (BOLA-318) already exists and reports
where BOLA and the registry disagree — read-only by design, repair is a
deliberate `bind` call. Run it before sizing any backfill; it answers in one call
what otherwise takes three hand-written DB queries.

⚠️ Reaching these from a laptop needs `kubectl port-forward`, and **a dead
tunnel answers `404 Cannot POST`, which is indistinguishable from "the route
does not exist"** — it nearly sent me hunting a phantom missing deploy. `curl
000` means the tunnel died; a 404 does not. Always probe a route you know exists
(`/v1/company/<id>`) in the same breath before concluding anything is missing.
