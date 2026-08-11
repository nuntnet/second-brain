---
name: reference-quota-no-allow-deny-rpc
description: "quota-management-backend has NO allow/deny RPC (verified from proto 2026-08-12) — only metering; any 'block when quota full' AC is unimplementable until BOLA-201 defines a contract"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T17:07:46.586Z
---

`backend/quota-management-backend` exposes exactly these RPCs (`src/interface/grpc_server/*/*.proto`, verified 2026-08-12):

- **quota_service**: `InternalCreateQuota`, `InternalUpgradeQuota`, `InternalListMyQuota`
- **assign_plan_service**: `InternalAssignQuota`, `InternalUsageAssignPlan`, `InternalRevokeAssignPlan`, `InternalRenewAssignPlan`, `InternalGetCurrentPlan`, `InternalCheckSummarize`, `InternalListTransactionOfQuota`

**There is no `canCreate` / `checkQuota` / allow-deny call.** It is usage **metering**, not a gate — see [[project-quota-not-feature-gate]].

**Why this keeps biting:** cards get written with ACs like "เช็ค quota ที่ server ก่อนสร้าง → 403 `quota_exceeded`" and "badge ใช้ไป X / Y … ไม่คำนวณเองใน CCS3" (BOLA-310 AC-03/05/09/13, BOLA-311, BOLA-312). Both are unimplementable as written: there's nothing to call for the gate, and the only reads (`InternalListMyQuota` / `InternalCheckSummarize`) would force the caller to compute remaining itself — which the AC forbids. BOLA-201 is the card that must define the contract (new RPC, or an explicitly-blessed read-then-decide pattern plus resource keys `workspace` / `workspace_member`); it is still To Do and its own text asks the same open question.

**How to apply:** when a card's quota AC requires server-side blocking, do NOT invent a client-side computation — that creates a second source of truth for limits, the same failure class [[project-bola-saas-access-model]] and BOLA-316 just cleaned up. Verify against the proto, flag the card, and treat it as a product/contract decision.
