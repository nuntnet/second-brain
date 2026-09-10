---
name: project_bola_binding_never_worked_via_ccs
description: "การผูก BOLA workspace อัตโนมัติตอนสร้างบริษัทพังทุก environment ตั้งแต่ย้ายมาสร้างผ่าน CCS (BOLA-317) เพราะ client ส่งแต่ X-System-Token ไม่ส่ง identity — แก้แล้ว 2026-09-10 (backoffice-api !559)"
metadata:
  type: project
---

**อาการ:** สร้างบริษัทใหม่แล้วไม่มี BOLA workspace แถว `oc2plus_bola_bindings` ค้าง `pending`
หน้าจอสมาชิกขึ้น empty state เหมือนยังไม่ได้ผูก LINE ทั้งที่ระบบควรผูกให้อัตโนมัติ

**สาเหตุที่ 1 (ตัวหลัก):** ตั้งแต่ย้ายการสร้าง workspace ไปผ่าน CCS
(`POST {CCS}/v1/companies/{id}/bola-workspaces`, BOLA-317) ตัว client
`bola_workspace_repository/ccs.go` ส่งแค่ `X-System-Token` · แต่ CCS route
`createBolaWorkspace` เรียก `helper.GetIdentityFromHeader` เป็นบรรทัดแรก ซึ่งอ่าน
`X-User-Id`/`X-User-Kind` ไม่มี = `ErrUnauthenticated` ก่อนถึงขั้นเช็ค permission ด้วยซ้ำ
→ **create ล้มทุกครั้ง ทุก environment** (ยืนยันกับ CCS local: ไม่ส่ง identity = unauthenticated,
ส่ง identity ของเจ้าของบริษัท = 201) · CCS authorize การสร้างกับบริษัทที่ workspace จะสังกัด
เจ้าของบริษัทจึงผ่านโดยไม่ต้องมี ops permission ส่วน **provider identity ถูกปฏิเสธ 403**

**สาเหตุที่ 2 (ทำให้มองไม่เห็น):** `bindBolaWorkspaceInternal` ที่ล้มขั้นดึงข้อมูลบริษัท return
ออกโดยไม่แตะแถว → `binding_attempts` ไม่ขยับ → `ReconcileStuckPending` (พลิก pending→failed
เมื่อ attempts ถึง 5) ไม่มีวันทำงาน → sweep ทุก 5 นาทีวนล้มเงียบตลอดอายุบริษัท

**แก้แล้ว (backoffice-api MR !559, merged 2026-09-10):** `BolaWorkspaceRepository.Create` รับ
`actor identity.Identity` เป็น argument แรกหลัง ctx · CCS client ส่ง `X-User-Id`/`X-User-Kind`
· bind ส่ง lookupIdent (เจ้าของตอนสร้าง / provider ตอน retry) · การล้มขั้น lookup นับ attempt แล้ว
· เทสต์ `TestCCS_Create_SendsTheCallerIdentity` กันการถอยกลับ

**ยังค้าง (ควรเป็นการ์ด):** retry sweep ไม่มี owner ให้ใช้ และ provider โดน 403 → binding ที่ล้ม
ตอนสร้างยังกู้เองไม่ได้ ต้องเก็บ owner ไว้บนแถว binding หรือหา owner ของบริษัทตอน retry

**Local (2026-09-10):** `.env` ของ backoffice-api ขาด `BOLA_SYSTEM_ADMIN_TOKEN` (ต้องเท่ากับ
CCS `BOLA_SYSTEM_TOKEN` และ BOLA `SYSTEM_ADMIN_TOKEN`) และ `CCS_SERVICE_BASE_URL` ชี้ผิดไป
`localhost:8085` (central-config) แทน `8092` (CCS) เติม/แก้แล้วทั้งคู่ · workspace ของ Nunt1/Nunt2
สร้างย้อนหลังผ่าน CCS ในนามเจ้าของ แล้ว UPDATE แถว binding เป็น active ด้วยมือ

เชื่อม [[project_oc2plus_liff_shell_is_the_line_entry]] [[reference_oc2plus_local_stack_recovery_traps]] [[project_bola_saas_access_model]]
