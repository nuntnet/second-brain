---
name: project_bola_binding_never_worked_via_ccs
description: "เส้นทางสร้าง BOLA workspace ผ่าน CCS ส่งแต่ X-System-Token ไม่ส่ง identity → CCS ตอบ unauthenticated เสมอ; แต่ CREATE_BOLA_WORKSPACE_VIA_CCS ปิดอยู่ทุก env จึงยังไม่กระทบ production — แก้แล้ว !559/!560 พร้อม retry ที่กู้เองได้ (2026-09-10)"
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

**แก้ให้ตัวเองตรงนี้ (สำคัญ):** ตอนแรกผมสรุปว่า "พังทุก environment" ซึ่ง**ผิด** · เส้นทาง CCS อยู่หลัง
feature flag `CREATE_BOLA_WORKSPACE_VIA_CCS` (`cmd/generics_server/main.go`, envDefault `false`) และ
**ไม่ได้ตั้งไว้ในไฟล์ values ของ env ไหนเลย** → dev/staging/production ยังสร้าง workspace ตรงไปที่ BOLA
(`BOLA_SERVICE_ENDPOINT` + system token) ซึ่ง config ครบ · บั๊ก missing-identity จึงเป็นระเบิดเวลา
สำหรับวันที่เปิด flag ไม่ใช่สาเหตุที่ production พัง

**สาเหตุจริงบนเครื่อง local:** `BOLA_SERVICE_ENDPOINT` ไม่ได้ตั้งใน `.env` และ **ค่า default ในโค้ดคือ
`http://localhost:8085`** ซึ่งบนเครื่องนี้คือ central-configuration-system → `POST /v1/workspaces` ได้ 404
ทุกครั้ง (BOLA local อยู่ 8097) · เติม `BOLA_SERVICE_ENDPOINT=http://localhost:8097` แล้ว sweep กู้ binding
กลับมา active ได้เองในรอบเดียว และใช้ workspace เดิมไม่สร้างซ้ำ

**เพิ่มเติมที่ทำใน !560:** retry ใช้ owner ที่เก็บไว้ (`owner_user_id`, migration member-api 015 merged),
adopt workspace ที่ CCS มีอยู่แล้วแทนการสร้างซ้ำ, และ response `customer-app` มี `binding_status`
(active/pending/failed) ให้หน้าจอสมาชิกแยกข้อความ "กำลังเตรียม" กับ "ล้มเหลว" (FE !591 merged)

**กับดักการดีบักที่เสียเวลาที่สุด:** log ของ service อ่านไม่ได้เพราะ process เป็น orphan (air ถูก launchd
รับเลี้ยง หลัง overmind session เดิมตาย) · ย้ายมารันใต้ overmind socket ใหม่ (`.overmind-oc2api.sock`,
`overmind start -l oc2plus-api -D`) แล้วถึงอ่าน pane ได้ และเห็น 404 จาก 8085 ทันที

