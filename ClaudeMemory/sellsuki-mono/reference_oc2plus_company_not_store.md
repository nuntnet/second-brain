---
name: reference_oc2plus_company_not_store
description: OC2Plus = company/บริษัท เท่านั้น ไม่มี store; Patona มี store จริงและคนละชั้น — ห้ามปนคำข้ามบอร์ด
metadata: 
  node_type: memory
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-09T16:08:43.826Z
---

🔴 **OC2Plus ไม่มี entity ชื่อ store** (verified 2026-08-09) — `store_id`/`StoreID` = **0** ทั้ง `3rdparty-api` และ `backoffice-api` ส่วน `company_id` = 191 / 779 · permission ผูก company ล้วน (`oc2plus.apikey.manage` บน `sellsuki.company:{id}`) · หน้าเลือกคือ `ChooseCompany.vue`

**"store" คือคำตกค้างจากยุคเก่าของ OC2Plus เอง** — เคยมี store service จริง (OC-1176/1177/1181), Choose store / Create store (OC-1281, OC-2173), user store preference (OC-1766/1767) · ย้ายมาใช้ CCS แล้วเปลี่ยนเป็น company **เฉพาะในโค้ด** การ์ด/copy ไม่ได้เปลี่ยนตาม การ์ดใหม่เลยลอกคำนำหน้า "[Store Admin]" ต่อ ๆ กันมา

⚠️ **อันตรายเพราะ Patona มี store จริง และคนละชั้น** — `space-go` 69 hits, `sellercenter-frontend` 150 hits ของ `store_id` (company มีได้หลาย store) · คนอ่านข้ามบอร์ด หรือ AI ที่ generate code จากการ์ด จะ implement ผิดชั้นทันที

**กติกาที่ user เคาะ 2026-08-09:**
| ระดับ | ใช้ |
|---|---|
| โค้ด / API / field / permission / ชื่อการ์ด | `company` |
| Thai copy ที่ผู้ใช้เห็น | **บริษัท** (ห้าม "ร้าน"/"ร้านค้า") |
| persona | **Company Admin** |

แก้แล้ว 11 การ์ด (OC-2269/2273/2274/4398/4425/4426/4427/4428/4429/4430/4431) + เพิ่ม DoD gate "ไม่มีคำว่า store โผล่ในโค้ด/copy/ชื่อ field" ทุกใบ · rule เต็มอยู่ที่ comment 43438 ของ OC-2275
⏳ ค้าง: description ของ OC-2275 เอง 4 บรรทัด (7, 9, 103, 160) · ซากใน `ChooseCompany.vue` (`.store-list`, `.text-store-name`, `alt="store"`)

**ที่จับได้เพราะ user ทัก** — ผมเขียน "ร้าน" ลงการ์ดใหม่ทั้งชุดโดยไม่เอะใจ · **ก่อนใช้คำเรียก tenant ในการ์ด/copy ให้ grep entity จริงในรีโปก่อน**

ดู [[project_oc2plus_3rdparty_apikey_gap]] · [[feedback_verify_absence_claims]]
