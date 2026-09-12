---
name: reference_oc2plus_otp_session_fails_3rdparty_consent
description: "OC2Plus web/OTP session has integration_id=='' → 3rdparty-api's consent lookup finds no row → 403 member_not_accept_consent on EVERY endpoint behind checkAuthenAndAuthroize, even for members who did consent"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 15cd28d0-899f-4256-bc49-d982e954e8ed
  modified: 2026-09-12T15:20:38.628Z
---

ทุก endpoint ของ 3rdparty-api ที่อยู่หลัง `checkAuthenAndAuthroize` เช็ค consent แบบนี้:

- `src/use_case/auth.go:21` → `getConsentStatus(ctx, ident.Id, sess.IntegrationID, ConsentTypeTOS)`
- `src/use_case/consent.go:48` → `consentIntegrationRepository.GetByID(ctx, integrationID, consentType)`

**consent ถูก key ด้วย `integration_id` (LINE integration) ไม่ใช่ member หรือ company.**

session ที่ member-api มินต์ให้ web/OTP login (OC-4348) มี `IntegrationID: ""` —
เขียนกำกับไว้เองที่ `src/use_case/auth.go:146-152` และเซ็ตที่ `:264` ว่า OTP session
ไม่มี LINE integration อยู่เบื้องหลัง → lookup ไม่เจอ row → `ConsentNotFound` →
**403 `member_not_accept_consent` ทุก request แม้สมาชิกยินยอมไปแล้วจริง**

ผลคือ **เส้น web/desktop ทั้งเส้นเรียก 3rdparty-api ไม่ได้เลย** ไม่ใช่แค่บาง endpoint
กระทบ OC-4348 / 4445 / 4446 / 4447 และเป็นเหตุผลหลักที่ Customer BFF เลิก proxy
(ดู [[project_oc2plus_customer_bff_reads_direct_not_proxy]])

**อาการที่หลอกตา:** LIFF/LINE session ผ่านปกติ เพราะมี integration_id จริง — ทดสอบด้วย
LINE แล้วเขียว แต่เว็บแดงหมด ดูเหมือน bug ของ session/cookie ทั้งที่เป็น consent lookup

**แก้ในรีโป member-api ไม่ได้** ต้องแก้ consent lookup ฝั่ง 3rdparty-api ให้ resolve
consent จาก member/company เมื่อไม่มี integration_id (หรือให้ OC-4345 เคาะ) — ยังไม่มี
การ์ดรองรับ ณ 2026-09-11 และเป็น prerequisite ของทุก endpoint ที่ยังต้อง proxy จริง
(redeem inquiry/confirm, address write) ใน [[project_oc2plus_customer_bff_reads_direct_not_proxy]]

## อัปเดต 2026-09-12 — อาการจริงคือ 500 ไม่ใช่ 403 แล้ว

ยืนยันสดในเบราว์เซอร์บน customer app (ดู [[reference_oc2plus_member_app_local_qa_session]]):
`integration_id=""` ถูกส่งเข้า query ที่ column เป็น uuid ตรง ๆ →
`pq: invalid input syntax for type uuid: ""` (`consent_integration_repository/postgresql.go:83`
เรียกจาก `consent.go:48`) → 3rdparty-api ตอบ **500 unexpected_error** →
member-api แปลงต่อเป็น **502** ให้ FE

แปลว่า **ไปไม่ถึง ConsentNotFound ด้วยซ้ำ** ไม่ได้ 403 อย่างที่จดไว้ตอนแรก —
ใครเสิร์ชด้วย "403 member_not_accept_consent" จะไม่เจอเคสนี้

ผลที่ผู้ใช้เห็น: แท็บ หน้าแรก / ของรางวัล / ประวัติ ขึ้น "ไม่สามารถโหลด...ได้"
ทั้งสามแท็บ เหลือแค่ โปรไฟล์ ที่ทำงาน (เพราะ `/me/profile` อยู่ใน member-api เอง
ไม่ผ่าน 3rdparty-api)

เกี่ยวข้อง: [[project_oc4348_web_otp_session_minter]] · [[project_oc2plus_consent_enforcement_model]] ·
[[reference_oc2plus_register_did_not_mint_a_session]]
