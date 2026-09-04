---
name: project_pat2658_reference_collision
description: "The QMS usage-trigger persistence work was filed under PAT-2658, an unrelated OMS 2.0 card. RESOLVED 2026-09-04: real card is PAT-2689; wrong number remains permanently in merged commits and in 8 source comments on main"
metadata:
  node_type: memory
  type: project
---

2026-09-04, `backend/quota-management-backend`.

I invented the number **PAT-2658** for the "persist the caller's usage trigger,
and return it" work without checking whether it was taken. It was:

> **PAT-2658 = "[OMS 2.0] Enhance schema เปิดรองรับหลายกลุ่มส่งมอบ (ยังไม่เปิดใช้งาน)"**, status To Do

Completely unrelated. Anyone following the reference lands on an OMS card.

Where the wrong number now lives — **commits are pushed, so they cannot be
amended** (monorepo rule):

- branch `feat/PAT-2658-persist-usage-trigger`
- commits `feat(PAT-2658): persist the caller's usage trigger, and return it`
  and `feat(PAT-2658): return plan_revision on GetCurrentPlan`
- QMS **MR !16** title
- Thai comments I posted on **AI-90** and **AI-196**

The neighbouring numbers ARE correct and should not be touched: **PAT-2657** =
billing period on `GetCurrentPlanResponse` + renewal CronJob; **PAT-2526** =
metering hooks (idempotent, overage-aware). Both Ready To Test.

**Resolved 2026-09-04.** The real card is **PAT-2689** — "[Quota][BE] Trigger
payload + idempotency_key อ่านกลับได้บน Transaction และ plan_revision บน
GetCurrentPlan". Correction comments posted on PAT-2689, PAT-2658, AI-196 and
AI-90.

Still outstanding, tracked as PAT-2689 AC5: **8 source files on QMS `main`
carry `PAT-2658` in comments** — `flow_context/transaction.go:43`,
`assign_plan_service.proto:190,299` (and the generated `.pb.go` mirroring
them), `dto_context.go:39`, `server.go:171`,
`transaction_repository/postgres_gorm_model.go:62`,
`internal_assign_plan.go:174`, `internal_usage_idempotency_test.go:258`.
Fix the `.proto` and regenerate; do not hand-edit `.pb.go`.

Also worth knowing for anyone tracing this: `b25ba92`/`58152e1` never merged
to `main` directly — they went into `feat/PAT-2526-usage-idempotency` (MR !16,
`17e6150`) and rode that branch up at `c23a9f9`. Searching the MR list finds a
different path than `git log main --grep`.

The general lesson, which is why this is written down: **check a Jira key exists
and matches before putting it in a branch name or commit message.** A wrong key
is unfixable once pushed and silently misroutes every reader afterwards.
