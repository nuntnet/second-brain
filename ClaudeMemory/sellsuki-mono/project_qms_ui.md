---
name: project_qms_ui
description: "QMS on CCS2 (epic OC-4257): Plan-centric reframe — Plan (commercial bundle, management-backend table) ≠ Quota (engine primitive) ≠ AssignPlan; MR !194/!7; quota Code≠PIS Reference is the recurring bug class"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

**QMS on CCS2 — epic OC-4257** · FE = `frontend/sellsuki-provider-management-frontend` (branch `feat/qms-admin-ui-oc4257`, **MR !7**) · BE proxy = `backend/management-backend` (branch `feat/qms-proxy-endpoints-oc4260`, **MR !194**) — QMS engine เป็น gRPC-only (50059) ไม่มี REST เอง

**Reframe ที่ PO เคาะ (2026-07-11): "Plan-centric — quota คือ building block"** — 3 ชั้นจริง:
- `Quota` = engine primitive (code+revision immutable + flow DAG) — งาน engineer, ซ่อนหลัง Advanced gate (flag `QMS_ADVANCED_QUOTA_AUTHORING_ENABLED` client-side, default off — OC-4373)
- `Plan` = **commercial bundle ใหม่ของ management-backend** (ตาราง `plan` + `plan_quota_item`) — provider admin CRUD รายวัน
- `AssignPlan` = engine ผูก quota→company · traceability ผ่านตาราง `plan_assignments` ฝั่ง management-backend (OC-4375)

**บั๊กประจำตระกูล: `quota.Code` ≠ `Balance.Reference` (PIS product code)** — ตย.จริง `bola_beta_free` ≠ `bola_workspace` · assign ใช้ Code, PIS picker ให้ Reference → silent-fail · แก้: picker ดึงจาก quota list (OC-4372) + auto-provision quota จาก product ด้วย **Code=product_code** (OC-4376 ensure-on-demand ใน GET /v1/qms/quotas — backfill ของเก่าฟรี)

**Engine gotchas (จาก review 2026-07-11):** quota revision เริ่มที่ 1 + engine exact-match — nil/"latest" ต้อง resolve ฝั่ง management-backend · RevokeAssignPlan revoke ทุก row ที่ (provider,user,code) ไม่มี revision filter → rollback ต้อง track per-item เอง · engine PK ถูกถือหลัง revoke (soft flag) → duplicate-PK ต้อง map เป็น already-assigned · assign แบบ atomic จริงทำไม่ได้ (gRPC นอก DB tx) = compensation + record-first pattern

**หน้า Products ของ provider-management = iframe ครอบ PIS app** — การสร้าง product ไม่ได้อยู่ใน repo นี้

Sprint 125 = ปิด QMS ทั้ง epic (17 ใบ code review + OC-4258 seed + OC-4263 gate + ใหม่ OC-4372/4373/4374/4375/4376) · commit ใหม่ทั้งหมด local บน 2 branch เดิม — push = เข้า MR !194/!7
