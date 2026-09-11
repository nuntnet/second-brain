---
name: reference_oc2plus_member_page_registry
description: "หน้าสมาชิกของ customer app มี registry ที่ backend เป็นเจ้าของ — CustomerAppPages() ใน backoffice-api/src/use_case/model/customer_app.go:48 ป้อนหน้า 'หน้าจอสมาชิก' (/member-screens) ที่แสดง slug + ลิงก์ + QR ให้แอดมิน; เพิ่ม route ใน member frontend เฉย ๆ ไม่พอ หน้าใหม่จะไม่โผล่ให้แอดมินเห็นเลย"
metadata:
  node_type: memory
  type: reference
---

**ค้นพบ 2026-09-11 เพราะ PO ชี้ไปที่ `https://oc2plus.sellsuki.local/member-screens`** ตอนกำลังสร้างหน้าใหม่ 6 หน้า — ผมไม่รู้ว่า registry นี้มีอยู่ และไม่ได้สั่ง agent ใดให้ลงทะเบียน

## registry อยู่ที่ backend ไม่ใช่ frontend

`backend/oc2plus-line-crm-service-backoffice-api/src/use_case/model/customer_app.go:48` → `CustomerAppPages() []CustomerAppPageDef`

คอมเมนต์ในไฟล์เขียนว่า *"the catalog, in the order the admin sees it. **Paths must match the member app's router (src/router/index.ts in the member frontend)**"* — **บรรทัดนั้นล้าสมัยตั้งแต่ OC-4500** เพราะ source of truth ย้ายไป `src/routeTable.ts` แล้ว

ประกาศไว้ 4 หน้า (ตรงกับที่มีอยู่จริงก่อนงานรอบนี้):
`/{slug}/register` (สมัครสมาชิก) · `/{slug}/login` (เข้าสู่ระบบสมาชิก) · `/{slug}/point-claims/new` (แจ้งใบเสร็จสะสมแต้ม) · `/{slug}/point-claims` (คำขอแต้มของฉัน)

## ทางที่ข้อมูลไหล

`GET /backoffice/v1/company/{companyId}/customer-app` (backoffice-api) → `RestCustomerAppResponse{ slug, pages[]{ key, label{th,en}, path, web_url } }`
→ frontend backoffice `src/services/customer-app/` → `src/views/MemberScreens/{MemberScreensView,CustomerAppCard}.vue` (route `/member-screens`, OC-4511)

`web_url` เป็น `""` ถ้า deployment ไม่ได้ตั้ง `MemberAppBaseURL` — หน้านั้นจะโชว์ `memberScreens.card.webMissing`

## ทำไมสำคัญ

หน้านั้นคือที่ที่แอดมิน **หา slug ของบริษัท** (`memberScreens.card.slug` = "รหัสบริษัทในลิงก์") และ **copy ลิงก์ / เปิด / ดาวน์โหลด QR ต่อหน้า** เพื่อส่งให้สมาชิก · ถ้าหน้าใหม่ไม่อยู่ใน registry **แอดมินไม่มีทางรู้ว่ามันมี และไม่มีทางส่งให้ลูกค้าเข้า** แม้ route จะทำงานปกติ

นี่ยังเป็นคำตอบของ "จะหา slug จริงมาเทสจากไหน" — endpoint `GET /v1/company/{slug}/theme` ของ member-api **คืน default ให้ทุก slug รวมถึงที่มั่วขึ้นมา** (ยืนยัน 2026-09-11: `zzz-definitely-not-a-real-slug-9f3a` ได้ 200 body เดียวกับ `oc2plus` เป๊ะ 286 bytes) จึง discover slug จากมันไม่ได้ · ส่วน endpoint ที่ resolve slug จริงเป็น POST ทั้งหมด (`auth/login`, `members/otp`, `members/verify`) ซึ่งยิงเล่นไม่ได้เพราะส่ง SMS OTP จริง

**ผลที่ใช้ได้ทันที:** หน้า login/register **render ได้ด้วย slug อะไรก็ได้** เพราะ theme คืน default → เทสงานหน้าตาได้เลยโดยไม่ต้องมี slug จริง (`/zzz-not-real/login` ใช้ได้) · ต้องมี slug จริงเฉพาะ flow ที่ยิง OTP

**How to apply:** เพิ่มหน้าใหม่ให้ customer app = แก้ 2 ที่ (`routeTable.ts` ฝั่ง member + `CustomerAppPages()` ฝั่ง backoffice-api) ไม่ใช่ที่เดียว · และ **แก้คอมเมนต์ที่ชี้ `src/router/index.ts` ให้ชี้ `src/routeTable.ts`** ตอนที่แตะไฟล์นั้น

เชื่อม [[project_oc2plus_liff_shell_is_the_line_entry]], [[project_oc4511_4514_ux_cluster]], [[project_oc2plus_member_react_migration]]
