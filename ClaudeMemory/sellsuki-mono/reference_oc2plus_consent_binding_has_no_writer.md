---
name: reference_oc2plus_consent_binding_has_no_writer
description: ตาราง consent (integration_id → consent_id) ไม่มีโค้ดที่ไหนเขียนเลยทั้ง monorepo — OC-4089 ยัง To Do; เกทอ่านตารางที่ไม่มีใครเติม และ key ผิดสำหรับเส้นเว็บ (OC-4545) · consent รายคนมีจริงแต่เก็บที่ consent service (Consentee keyed ด้วย reference_id) ไม่ใช่ใน CRM — และ API เป็น point lookup ไม่มี list จึงกรองเป็นชุดไม่ได้ถ้าไม่ consume Kafka
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-13T15:50:13.742Z
---

ตรวจ 2026-09-13 ตอนพยายาม QA กระเป๋าคูปองบน dev

## chain มี 3 ท่อน ขาดท่อนกลาง

| ท่อน | เจ้าของ | สถานะ |
| --- | --- | --- |
| เอกสาร consent ใน `sellsuki-service-consent` | OC-3897 | ✅ Done (dev มี 50 ใบ) |
| **binding "บริษัทนี้ใช้เอกสารใบไหน" = ตาราง `consent`** | **OC-4089** | 🔴 **To Do — ไม่มีโค้ดเลย** |
| สมาชิกกดยอมรับ | OC-4340 / OC-4361 | ✅ (engine `AcceptConsent` + FE `WebConsentGate`) |

`consent_integration_repository` มีแค่ `GetByID` — **grep ทั้ง 3 service ไม่มี INSERT/UPDATE/DELETE ตาราง `consent` เลย**
ทุกแถวที่มีอยู่จริงเป็นของที่คนใส่มือ → บริษัทที่ไม่มีใคร INSERT ให้ = ถูกบล็อกทั้งแอป

## key ผิดสำหรับเส้นเว็บ → **OC-4545**

`consent` key ด้วย `integration_id` (LINE OA) · สมาชิก web/OTP ไม่มี integration ·
member-api workaround ที่ `integration_repository/postgresql.go:267` ต้องมี active **พอดี 1 ใบ**
→ 0 ใบพัง, 2+ ใบก็พัง · resolve ไม่ได้ → `""` เข้า column uuid →
**500 `unexpected_error`** (`consent_integration_repository/postgresql.go:83`) ทุก `/me/*`
ดูเหมือน service ล่มทั้งที่เป็นแค่ config ว่าง

## path จริงของ engine (เคยยิงผิดมาแล้ว)

`/v1/consent/{type}/status` · `/v1/consent/{type}` · `/v1/consent/{type}/accept` — **ไม่มี `/me/` นำหน้า**
(`/me/` เป็นของฝั่ง BFF member-api) · identity kind ที่รับคือ `user` / `oc2plus.user` เท่านั้น
(`oc2plus.member` → 401 `user_type_not_allowed`)

## fixture บน dev (ต้องลบเมื่อ OC-4545 เสร็จ — AC-06)

dev มี integration ใบเดียวเป็นของบริษัทที่ไม่มี member · มี member คนเดียวอยู่บริษัทที่ไม่มี integration
→ ใส่ fixture ให้บริษัท `a3fa1608-…` (slug `d9pesc1nm7id80npgljg`):
- integration `11111111-1111-4111-8111-000000004526` ชื่อ `FIXTURE consent unblock (OC-4526)`
- `consent` 2 แถว ยืมเอกสารที่ active อยู่แล้ว: pdpa→`100`, tos→`83`
แล้ว accept ทั้งสองใบผ่าน `POST /v1/consent/{type}/accept` → `/me/coupons` ตอบ 200

**How to apply:** ถ้าเจอ 500 `unexpected_error` บนทุก `/me/*` ของ OC2Plus อย่าไปไล่หาบั๊กใน endpoint นั้น —
เช็ค `integration` + `consent` ของบริษัทนั้นก่อน

เชื่อม [[project_oc2plus_consent_enforcement_model]] [[reference_oc2plus_otp_session_fails_3rdparty_consent]] [[project_oc4526_coupon_wallet]] [[reference_oc2plus_member_app_local_qa_session]]

## "กรองตาม consent ของสมาชิก" — ข้อมูลมีจริง แต่รูปแบบไม่เหมาะกับการกรองเป็นชุด (ตรวจ 2026-09-16)

🔴 **ผมเคยเขียนหัวข้อนี้ผิด** ว่า "ไม่มี consent รายคนเลย" · PO ทักว่า *"ที่ consent service
น่าจะมีที่เก็บ consent ราย member นะ"* — **ถูกต้อง** ผมตรวจแค่ฐาน `oc2plus_crm` แล้วสรุปเลย
เถียงข้ามขอบเขตที่ตัวเองตรวจ ซึ่งเป็นความผิดพลาดแบบเดียวกับที่
[[feedback_grep_the_spec_is_not_grep_the_service]] บันทึกไว้

คนละคำถามกับหัวข้อข้างบน ข้างบนคือ *บริษัทนี้ผูกเอกสารใบไหน* ส่วนข้อนี้คือ
**ใครยอมรับ/ถอนความยินยอมบ้าง** ซึ่งเป็นสิ่งที่ฟีเจอร์ประเภท export / feed / report
ต้องใช้ และ **ไม่มีอยู่ใน `oc2plus_crm` เลย**

**ของจริงอยู่ที่ `backend/sellsuki-service-consent`** — `src/entity/consentee/consentee.go`
เก็บ `Consentee{ID, ConsentID, Version, ReferenceID, IPAddress, UserAgent, MergedFrom/To}`
คือหนึ่งแถวต่อ (เอกสาร consent × คน) โดย `ReferenceID` เป็น key รายคน · ค่าใน option เป็น
`"accepted"` / `"declined"`

**สิ่งที่ไม่มีใน CRM (ซึ่งยังจริง):** ไม่มีตาราง `member_consent` ไม่มีคอลัมน์ consent บน
`member` · สองตารางที่ชื่อ consent ใน `oc2plus_crm` เป็นระดับบริษัท/OA · ฝั่ง OC2Plus เรียก
ได้จาก `member-api/src/repository/consent_repository/rest.go` (`GetConsenteeByRefID`,
`UpdateDPAConsenteeStatus`) เท่านั้น — backoffice-api มีแค่ `company_consent_repository`

### ข้อจำกัดจริงคือรูปแบบ API ไม่ใช่การมีอยู่ของข้อมูล

ทั้ง service มี **7 path** เท่านั้น (`/consent`, `/consent/{id}`, `/consent/{id}/preview`,
`/consent/{id}/render`, `/consentee`, `/consentee/merge`, `/cache`)

- `GET /consentee` บังคับทั้ง `consent_id` **และ** `reference_id` → **point lookup**
- **ไม่มี list/search consentee เลย** · repository ก็ get/set/delete ด้วย
  (consentID, version, referenceID) เท่านั้น
- แปลว่า export N แถว = **N ครั้งของการยิงรายคน** ไม่ใช่ join

- **ไม่มีคำว่า withdraw / revoke / opt-out ในโค้ดเลยสักที่** — "ถอนความยินยอม" ไม่ใช่สถานะ
  ของเรคอร์ด แต่เป็น option ที่ถูก PUT ให้เป็น `declined` · การ์ดที่เขียนว่า "member ที่ถอน
  consent" จึงต้องนิยามใหม่เป็น "option = declined"

### ทางที่สามที่ผมมองข้ามตอนแรก: Kafka

`src/repository/consentee_event_repository/kafka.go` publish `CreateEventConsentee` และ
`UpdateEventConsentee` ออกไปทุกครั้งที่ consentee ถูกสร้าง/แก้ → **consume แล้วทำ read model
ของตัวเองได้** ซึ่งเป็นวิธีที่ทำให้ feed กรองเป็นชุดได้จริง โดยไม่ต้องยิงรายคน · ไม่ได้อยู่
ในขอบเขต OC-4479 v1 แต่เป็นคำตอบของ Phase 2

⚠️ แก้ข้อความข้างบนด้วย: ตาราง `consent` บน local **มี 4 แถว** และ `company_consent` มี 2
(ตรวจ 2026-09-16) — "ไม่มีใครเขียน" ยังจริงในแง่ไม่มี INSERT ในโค้ด แต่ไม่ได้แปลว่าตารางว่าง

**ผลที่เกิดขึ้นจริง:** OC-4479 (sale-transaction feed) เขียน AC ว่า "ห้ามส่ง member ที่ถอน
consent" · PO เคาะ 2026-09-16 ว่า **v1 ตัด consent filter ออก เหลือ pseudonymization**
⚠️ **การตัดสินใจนั้นตั้งอยู่บนคำอธิบายของผมที่ผิด** ("ไม่มีอะไรให้กรอง") เหตุผลที่ถูกคือ
*กรองได้ แต่ต้องยิงรายคน N ครั้งต่อการ export หนึ่งรอบ และ "ถอน" ไม่ใช่สถานะที่ service
โมเดลไว้* — ข้อสรุปเดิมยังใช้ได้สำหรับ v1 แต่ PO ควรได้รู้เหตุผลที่ถูกต้อง

**How to apply:** ถ้าการ์ดไหนสั่งให้กรอง/ซ่อนข้อมูลตาม consent ของสมาชิก ให้ตอบทันทีว่า
ต้องเลือกก่อนว่าจะ (ก) ให้ service นั้นต่อ consent service เอง (ข) ย้ายงานไป member-api
หรือ (ค) ตัด filter ออกจาก v1 — อย่าเริ่มเขียนโดยสมมติว่ามี join ให้ทำ

---

# ✅ ล้าสมัยแล้ว 2026-09-22 — มี writer แล้ว แต่ยังไม่ถึง staging

ท่อนกลางที่เคยขาดถูกสร้างแล้ว ตรวจบน `origin/develop`:

- **writer**: backoffice-api `PUT /v1/company/{id}/consent-surfaces/{surface_id}`
  → `company_consent_repository` (`const table = "company_consent"`)
- **reader**: 3rdparty-api `resolveConsentBinding` → member → company →
  `companyConsentRepository.GetByCompany` แล้วค่อย fallback ไป integration เดิม
  (OC-4545) · wire อยู่บน **main** แล้ว
- **FE**: backoffice หน้า `/consent-surfaces` — mock ถูกลบทิ้ง ไม่ใช่แค่ปิดสวิตช์
  (`services/consent-surface/index.ts` export ตัวจริงตัวเดียว)

🔴 **แต่ OC-4089 อยู่แค่ `develop` ทั้งสองฝั่ง** — `git grep` บน `origin/main` ของ
backoffice-api และ backoffice FE ได้ 0 ทั้งคู่ ⇒ **บน staging ตั้งค่า consent
ไม่ได้เลย ไม่มีทั้งหน้าจอและ API** ส่วน 3rdparty-api ฝั่งอ่านขึ้น main แล้ว
⇒ staging อ่านได้ แต่ไม่มีทางเขียน

อาการที่ตามมา: สมาชิกบน staging เจอ `consent_not_configured` แล้วแอดมินแก้เองไม่ได้
ต้องเอา OC-4089 ขึ้น main ก่อน (cherry-pick จาก main เหมือนที่ทำกับ OC-4587) หรือ
ทดสอบบน dev แทน

ดู [[reference_member_api_service_urls_were_never_set]] ·
[[project_oc4089_consent_binding_page]]
