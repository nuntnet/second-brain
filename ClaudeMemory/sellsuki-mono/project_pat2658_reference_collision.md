---
name: project_pat2658_reference_collision
description: "The QMS usage-trigger persistence work was filed under PAT-2658, which is actually an unrelated OMS 2.0 schema card — wrong number is now in merged commits, MR !16's title and Jira comments on AI-90/AI-196; unresolved"
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

**Still unresolved** — the user was asked whether to open a correctly numbered
card and post correction comments, and has not answered. Do not renumber
unilaterally: the board is theirs
([[feedback-report-wrong-cards-dont-edit]]).

The general lesson, which is why this is written down: **check a Jira key exists
and matches before putting it in a branch name or commit message.** A wrong key
is unfixable once pushed and silently misroutes every reader afterwards.
