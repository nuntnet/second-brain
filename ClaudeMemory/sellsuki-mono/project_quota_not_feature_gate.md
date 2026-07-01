---
name: project_quota_not_feature_gate
description: quota-management-backend เป็น usage-based metering ไม่มี boolean feature-gate/hasFeature() capability
metadata: 
  node_type: memory
  type: project
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

`backend/quota-management-backend` เป็น **usage-based metering engine** (AssignPlan, balance = `Product{Quantity}`, debit ผ่าน UsageFlow DAG; RPC: InternalGetCurrentPlan/CheckSummarize/UsageAssignPlan). **ไม่มี `hasFeature(company)` / boolean capability RPC** และ data model ไม่มี concept "feature on/off ต่อ company".

**How to apply:** ถ้า card เขียน guard แบบ `hasFeature(company_id, "X")` / "quota/plan guard" สำหรับ feature-gate (เปิด/ปิดฟีเจอร์ต่อ company) = ผิด contract — quota ตอบ boolean capability ไม่ได้. feature gate ต้องเลือก: (ก) plan/subscription includes-flag, (ข) capability/feature-flag store ใหม่ (card แยก), หรือ (ค) Admin override + Keto server-side (Phase 1). เจอใน PAT-2425 → user เลือก (ค) admin override.

เกี่ยวกับ [[project_qms_ui]].
