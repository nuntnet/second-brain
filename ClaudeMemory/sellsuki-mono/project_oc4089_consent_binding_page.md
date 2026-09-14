---
name: project_oc4089_consent_binding_page
description: OC-4089 หน้าผูกเอกสาร consent — FE ทำ mock+route แล้ว (MR !613); enforcement อยู่ที่ option ไม่ใช่ document; ตาราง company_consent รับแค่ pdpa/tos และไม่มีคอลัมน์ enforcement_mode
metadata: 
  node_type: memory
  type: project
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-14T12:00:12.984Z
---

2026-09-14 ทำหน้า "เอกสารยินยอมหน้าสมัครสมาชิก" ที่ `/member-registration/consent`
บน backoffice FE (MR !613, branch `feat/oc-4089-consent-binding-mock`) — **mock อย่างเดียว**
เพราะ backoffice-api ไม่มี route consent เลยสักตัว สลับเป็นของจริงด้วย
`VITE_CONSENT_BINDING_MOCK=false` (contract อยู่ใน `src/services/consent-binding/real.ts`)

**สามข้อที่ต้องรู้ก่อนทำ OC-4089 ต่อ:**

1. **enforcement อยู่ที่ OPTION ไม่ใช่ DOCUMENT** — `sellsuki-service-consent` เก็บ
   `ConsentOption.type` ∈ `require_accept|acknowledge_optional|display_only`
   ตัว `Consent` entity **ไม่มีฟิลด์ enforcement** ดังนั้น `doc.enforcement` ที่การ์ด
   OC-4089/OC-3897 อ้างถึง **ยังไม่มีอยู่จริง** ต้อง derive จาก option

2. **envelope จริงอยู่ที่ `src/use_case/update_consent_status.go:35-41`**
   (`typeEnforcementAllowList`) — pdpa/tos = require|optional · marketing = optional
   เท่านั้น · cookie = optional|display · privacy_notice = display เท่านั้น
   โหมดนอกนี้ถูกปฏิเสธด้วย `invalid_enforcement_for_type` (400)

3. **⚠️ สกีมา CRM รองรับได้แค่ครึ่งเดียวของการ์ด** — ตาราง `company_consent`
   (member-api `migrations/020`) มี `CHECK (consent_type IN ('pdpa','tos'))`,
   PK = `(company_id, consent_type)` (หนึ่งประเภทหนึ่งใบ) และ **ไม่มีคอลัมน์
   `enforcement_mode`** → marketing/cookie/privacy_notice + mode + "เลือกได้ 0–N"
   ต้องมี migration ก่อน · ถ้าจะส่งเร็ว ตัด scope เหลือ pdpa+tos ทำได้เลยโดยไม่แตะสกีมา

permission เขียนระดับ Company Owner ยังไม่มีใน Keto catalog หน้าจึงเกทด้วย
`MEMBER_VIEW` ไปก่อน · ลิงก์ "ไปตั้งค่า" ของแถว consent ใน checklist
[[project_oc4551_readiness_checklist]] ยังไม่ต่อมาหน้านี้จนกว่าจะบันทึกได้จริง

ดู [[reference_oc2plus_consent_binding_has_no_writer]] และ
[[project_oc2plus_consent_enforcement_model]]
