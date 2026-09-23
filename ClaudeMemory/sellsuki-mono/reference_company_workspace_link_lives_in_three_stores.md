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

`slug` is duplicated across all three too. Staging counts: **3 / 7,514 / 6,857** —
no two agree.

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
