---
name: reference_oc2plus_consent_binding_has_no_writer
description: ตาราง consent (integration_id → consent_id) ไม่มีโค้ดที่ไหนเขียนเลยทั้ง monorepo — OC-4089 ยัง To Do; เกทอ่านตารางที่ไม่มีใครเติม และ key ผิดสำหรับเส้นเว็บ (OC-4545)
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
