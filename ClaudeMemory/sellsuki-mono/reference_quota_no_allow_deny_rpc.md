---
name: reference-quota-no-allow-deny-rpc
description: "quota-management RPC surface: no allow/deny (built in unmerged MR !13), and usage write has no dimensions/no idempotency key, no billing period to read"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-12T17:32:22.696Z
---

`backend/quota-management-backend` exposes exactly these RPCs (`src/interface/grpc_server/*/*.proto`, verified 2026-08-12):

- **quota_service**: `InternalCreateQuota`, `InternalUpgradeQuota`, `InternalListMyQuota`
- **assign_plan_service**: `InternalAssignQuota`, `InternalUsageAssignPlan`, `InternalRevokeAssignPlan`, `InternalRenewAssignPlan`, `InternalGetCurrentPlan`, `InternalCheckSummarize`, `InternalListTransactionOfQuota`

**There is no `canCreate` / `checkQuota` / allow-deny call.** It is usage **metering**, not a gate — see [[project-quota-not-feature-gate]].

**Why this keeps biting:** cards get written with ACs like "เช็ค quota ที่ server ก่อนสร้าง → 403 `quota_exceeded`" and "badge ใช้ไป X / Y … ไม่คำนวณเองใน CCS3" (BOLA-310 AC-03/05/09/13, BOLA-311, BOLA-312). Both are unimplementable as written: there's nothing to call for the gate, and the only reads (`InternalListMyQuota` / `InternalCheckSummarize`) would force the caller to compute remaining itself — which the AC forbids. BOLA-201 is the card that must define the contract (new RPC, or an explicitly-blessed read-then-decide pattern plus resource keys `workspace` / `workspace_member`); it is still To Do and its own text asks the same open question.

**How to apply:** when a card's quota AC requires server-side blocking, do NOT invent a client-side computation — that creates a second source of truth for limits, the same failure class [[project-bola-saas-access-model]] and BOLA-316 just cleaned up. Verify against the proto, flag the card, and treat it as a product/contract decision.

---

## UPDATE 2026-08-13 — built, in quota-management MR !13 (branch `feat/bola-201-quota-check-rpc`, unmerged)

The deeper finding: **the flow engine could not express a verdict at all.** `FatalNode`,
`SuccessNode` and `RollbackNode` each returned `("", nil)` from `Process` and
`Flow.Run` returned `nil` for all three, so "the quota refuses" and "the quota
allows" were the same answer — and `InternalUsagePlan` committed its transaction
either way. So the limit did not exist even on the spending path, silently.
`Quota.RunEligibilityFlow` existed in the entity and **nothing called it**.

What !13 adds:
- `flow_context.Outcome` (`success`/`fatal`/`rollback`/`""`) set by the three
  terminal nodes; `Allowed()` is true ONLY on explicit success — undetermined must
  never read as permission, or a misconfigured quota is the most permissive object
  in the system.
- `Quota.DryRun` — eligibility flow if defined, else a dry run of the usage flow,
  on **copies** of balance/state (the flows mutate what they are given; that is how
  usage debits, so sharing the caller's pointer makes asking equal acting).
- `InternalCheckAssignPlan` RPC + `UseCase.InternalCheckPlan` — no lock, no
  transaction, no write. Same `MatchAssignPlan` as usage so the answer comes from
  the plan that would be charged.
- `FlowContext.outcome` on the wire, so existing usage callers can SEE a fatal.

**Limits to remember before using it:** the answer is a snapshot, not a
reservation — two callers asking about the same last unit both get yes. Anything
that must not be double-spent still has to be enforced where it is spent.

**Still undecided (needs a human):** should `InternalUsagePlan` refuse to commit
when the flow ends at a fatal node? That is the real enforcement gap, but it
changes live behaviour for existing Patona callers.

A `workspace_member` quota SKU must also exist in data before BOLA-187/194/196's
"block when full" ACs can be wired — that is product/ops, not code.


---

## UPDATE 2026-08-28 — QMS ยัง "รับ usage แบบมี dimension" และ "บอกรอบบิล" ไม่ได้ด้วย

ตรวจ proto รอบใหม่ตอนรีวิว epic AI-6 (E5 LLM Metering) เจออีก 3 ช่องที่คนละเรื่องกับ allow/deny:

- **`UsageAssignPlanRequest{api_key, provider, user, data}`** — dimension ทุกตัวยัดได้แค่ใน `data`
  ที่เป็น **string ทึบ** ⇒ ถาม QMS ว่า "workspace/model/caller_service ไหนใช้เท่าไร" ไม่ได้เลย
- **ไม่มี idempotency key บน usage write** ⇒ replay จาก durable spool หลัง process ตาย = นับซ้ำจริง
  กันได้แค่ในบัฟเฟอร์ฝั่ง caller ไม่ใช่ end-to-end
- **`GetCurrentPlanResponse` คืน `FlowContext` เปล่า** (`ContextValue` + `Event[]` + `gas_used`) —
  ไม่มีวันเริ่ม/วันสิ้นสุดรอบบิล และ renewal เป็น flow ที่ต้องมีคนเรียก `InternalRenewAssignPlan`
  ⇒ "รีเซ็ตโควตาเองเมื่อขึ้นรอบใหม่" และ `resetAt` บน UI ไม่มีแหล่งข้อมูล
- ที่ใกล้ read-model ที่สุดคือ `InternalListTransactionOfQuota` ซึ่งคืน `Transaction` ดิบทีละรายการ
  (`quota_code`/`plan_revision`/`user_ref`/`provider_ref`/`occurred_at` + `Event[]`) — ไม่มี group-by

**How to apply:** อย่าออกแบบให้ QMS เป็นที่เก็บ usage แบบ analytics · เส้นแบ่งที่เคาะแล้ว
(แผน §5.14.3 D3, 2026-08-28): **QMS ถือหน่วยที่ enforce + billing period · breakdown/cost อยู่ฝั่ง
ผู้เรียก** · ถ้าการ์ดไหนเขียนว่า "ห้ามมี store ของตัวเอง อ่านจาก QMS อย่างเดียว" ให้ถือว่า
implement ไม่ได้ และต้องแก้การ์ดนั้น ไม่ใช่ฝืนทำ
