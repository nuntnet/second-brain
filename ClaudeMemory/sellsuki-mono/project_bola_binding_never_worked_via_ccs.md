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

## 🔴 แก้ความเข้าใจผิดอีกข้อ (2026-09-13): **ไม่มี sweep ทุก 5 นาที — ไม่มีใครเรียก RetryPendingBindings เลย**

ที่เขียนไว้ข้างบนว่า "sweep ทุก 5 นาทีวนล้มเงียบ" **ผิด** · ตรวจจริงแล้ว:
`cmd/generics_server/main.go` ของ backoffice-api **ไม่มี ticker/goroutine** เรียก `RetryPendingBindings`
(grep `RetryPendingBindings|Ticker` = 0) · ทางเดียวที่เรียกได้คือ `POST /v1/system/bola-bindings/retry`
ซึ่ง commit `b038aa6` เขียนเองว่า *"for manual cron trigger"* — และ **CronJob นั้นไม่เคยถูกสร้าง**:
`deployment/values-*.yml` ไม่มี cron block, บน cluster `octoplus-dev` มีแค่ `cron-cleanup` กับ `cron-daily-report`

**ผล:** บริษัทที่ bind รอบแรกไม่ผ่าน ค้าง `pending` ตลอดอายุบริษัท **ทุก env** → ลิงก์แอปสมาชิกตายถาวร
policy ทั้งชุด (`ReviveStalledFailed` cooldown 1 ชม., `ReconcileStuckPending` cap 5 attempts) เขียนไว้แต่ไม่มีวันทำงาน

**หลักฐานจาก dev 2026-09-13** — บริษัท `d9fca606-aea6-4ef3-b10c-4a0a8dcf0dcd`:
`GET /v1/system/company/{id}/customer-app` (ยิงจากใน pod ด้วย `$SYSTEM_TOKEN`) ตอบ
`{"binding_status":"pending","slug":null,"pages":[]}` ขณะที่เส้นของแอดมิน (Kratos identity) คืน slug
`d9tefkcjauae8rpaarm0` = `company.Code` จริง → **system endpoint คืน slug null เพราะ lookup ด้วย provider
identity ซึ่ง CCS หาไม่เจอ** (เมนู LINE ที่ BOLA สร้างจากเส้นนี้จะไม่มี slug — OC-4514)

**ชั้นที่สองที่เพิ่งเจอ:** backoffice `customer_app.go:61-68` ปลด slug ออกจาก BOLA แล้ว (คอมเมนต์ยาวว่า
"web links work in any browser") แต่ member-api `ResolveCompanyBySlug` ยัง resolve ผ่าน
`oc2plus_bola_bindings.GetBySlug` ตัวเดียว และ `CompanyRepository` ของ member-api มีแค่ `GetCompanyDetail(id)`
ไม่มี lookup ด้วย code → **สองบริการไม่ตรงกันว่า slug อยู่ที่ไหน** · การ์ด [[OC-4544]]

**How to apply:** ถ้าเจอ member-api ตอบ 404 ที่ `/v1/company/{slug}/...` ทุก slug — อย่าไปหาที่ route
ให้เช็ค `binding_status`/`slug` ของบริษัทนั้นก่อน · แยก "route ไม่มี" ออกจาก "resolve ไม่เจอ" ด้วย body:
Fiber ตอบ `Cannot POST /x` เมื่อ route ไม่มี แต่ตอบ envelope `{"error_code":"not_found"}` เมื่อ handler ทำงานแล้ว
