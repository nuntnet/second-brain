---
name: project_oc2plus_customer_bff_reads_direct_not_proxy
description: "เคาะ 2026-09-11: Customer BFF ฝั่ง read อ่าน oc2plus_crm ตรง ไม่ proxy ไป 3rdparty-api — MR !109 merge, !105 (OC-4347) ถูกทิ้ง, !106 ยังเหลือ 5 endpoint ที่ต้อง retarget"
metadata: 
  node_type: memory
  type: project
  originSessionId: 15cd28d0-899f-4256-bc49-d982e954e8ed
  modified: 2026-09-11T16:46:47.241Z
---

`project_customer_app_program` บันทึกไว้ว่า "member-api = Customer BFF — browser ห้ามยิง
3rdparty-api ตรง" — ข้อนั้นยังจริง **แต่วิธีที่ BFF หาข้อมูลเปลี่ยนแล้ว**

**เคาะ 2026-09-11: endpoint ฝั่ง read อ่านตาราง `oc2plus_crm` ตรงจาก member-api ไม่ proxy ผ่าน 3rdparty-api**

merge แล้ว: MR !109 (OC-4538) → `develop` ที่ `38fa30f` — `/me/point`,
`/me/point/transaction`, `/me/point/expire`, `/me/campaigns`

**ทิ้ง: MR !105 (OC-4347) `feat/oc-4347-customer-bff`** — proxy 4 endpoint ไป 3rdparty-api
เขียนเสร็จ CI เขียว แต่ไม่ merge

เหตุผลที่ proxy แพ้ (เรียงตามน้ำหนัก):
1. **proxy ตอบ 403 ทุก request บน session ของ FE ที่ ship ไปแล้ว** — FE ล็อกอิน OTP บนเว็บ
   ดู [[reference_oc2plus_otp_session_fails_3rdparty_consent]] · ข้อนี้ข้อเดียวก็จบ
2. รูป response ของ campaign ใน !105/!106 (`MeCampaignSummary`: `code`/`image`/`point_cost`/
   `condition_type` ไม่มี `remaining_quota`) **ไม่ตรงกับ type ที่ FE merge ไปแล้ว**
   (`CampaignListItemDto`: `campaign_code`/`image_url`/`cost_point`/`display_flag`/
   `remaining_quota`/`category_code`) ส่วน !109 ตรงทุก field
3. ไม่ผิดกฎ boundary — 3rdparty-api ต่อ `POSTGRES_CRM_DB_NAME` ตัวเดียวกับ member-api
   ตาราง shared by design และ `.claude/rules/oc2plus-service-boundary.md` บอกว่า boundary
   คือ actor ไม่ใช่ domain → actor เป็น member = member-api เป็นเจ้าของ

**ราคาที่ยอมจ่าย:** !109 เขียน logic ประเมิน campaign ซ้ำ ~300 บรรทัด และจงใจต่างจาก
upstream 3 จุด (ซ่อน campaign `suspended` เพราะ FE ไม่มีค่านั้นใน union · นับ batch ที่ไม่มี
วันหมดอายุเข้า balance · นับ total ก่อน limit/offset) มีคอมเมนต์กำกับที่ call site ทุกจุด
→ **สองเซอร์วิสตอบ "แลกอะไรได้บ้าง" ไม่เหมือนกันได้** ต้องคุมไม่ให้เบี่ยงเพิ่ม

## ค้างอยู่ (ณ 2026-09-11)

- **!105 ยัง open** ต้องปิดโดยไม่ merge — ถ้าปล่อยไว้แล้วมีคน merge จะชน `/me/point` +
  `/me/campaigns` ที่อยู่บน develop แล้ว
- **!106 (wave 2) อย่าปิด** มี 5 endpoint ที่ !109 ไม่มี: `/me/campaigns/{code}`,
  address CRUD + `/primary`, `/me/consent/{type}/status`, redeem inquiry/confirm ·
  ตอนนี้ stack อยู่บน !105 → ต้อง retarget เป็น `develop` และ reshape campaign detail
  ให้ extend `MeCampaignListItem` ของ !109 (FE เขียน `CampaignDetailDto extends CampaignListItemDto`)
- **redeem / address write เลี่ยง 3rdparty-api ไม่ได้** (write + business logic) →
  ติดกำแพง consent เหมือนเดิม ต้องแก้ที่ 3rdparty-api ก่อน
- contract ที่ยังขาดตามคอมเมนต์ OC-4347 (2026-09-10): `terms` แยกจาก `description`,
  `fulfilment_method` pickup/delivery (block OC-4354), coupon artifact หลัง redeem ·
  `category_code` ได้มาแล้วจาก !109

**หมายเหตุกระบวนการ:** !105 กับ !109 สร้าง endpoint ชุดเดียวกันคนละแบบ โดย MR body ของ !109
เขียนว่า "the endpoints did not exist in this service on any branch" ซึ่งไม่จริง — เคสซ้ำของ
[[feedback_grep_the_spec_is_not_grep_the_service]] แก้ข้อความใน MR แล้วตอน merge

เกี่ยวข้อง: [[project_customer_app_program]] · [[project_loyalty_point_cluster]] ·
[[project_oc2plus_member_react_migration]]
