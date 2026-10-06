---
name: reference_local_3rdparty_consent_acceptance_off
description: บน local การกดยอมรับ consent ตอนสมัครได้ 503 CONSENT_UNAVAILABLE เพราะ 3rdparty-api ไม่มี CONSENT_ACCEPTANCE_ENABLED (default false) — ไม่ใช่ bug
metadata:
  type: reference
---

(ตรวจ 2026-10-06) สมัครสมาชิกบน local ผ่าน OTP + โปรไฟล์ได้ แต่ขั้นยินยอม `PUT /v1/me/consent-policy/acceptance` ได้ **503 `CONSENT_UNAVAILABLE`** ทุกครั้ง

- ต้นเหตุ: 3rdparty-api (`:8103`) รันโดยไม่มี env `CONSENT_ACCEPTANCE_ENABLED` → default `false` → `consent_policy_acceptance.go:34` ตอบ unavailable ทันที ไม่ได้เรียก consent service เลย (log ของ consent service จะเห็นแค่ GET)
- dev / staging ตั้ง `true` ใน `deployment/values-*.yml` แล้ว → บน cluster ไม่เป็น
- แก้บน local = สตาร์ต 3rdparty-api ใหม่โดยมี env นั้น (แบบเดียวกับชี้ member-api ไป stub: env นำหน้า `overmind start`) — แต่ถ้า socket ของ overmind ตัวนั้นถูก instance อื่นทับ (`-s .overmind.sock` ซ้ำ) จะคุมไม่ได้ ต้องให้เจ้าของ stack จัดการ ห้าม kill process เอง
- ข้อมูลประกอบ: บริษัทที่ไม่มี `company_consent` เลย ขั้นยินยอมจะขึ้น "ยังไม่ได้ตั้งค่าเอกสารสมาชิก" ปุ่มกดไม่ได้ · เอกสารทดสอบที่ publish แล้วบน local: pdpa `434403`, tos `434404` (Mongo `develop-sellsuki-service-consent.consent`)
- สมาชิกถูกสร้าง (`POST /members` 201) **ก่อน** ขั้นยินยอม — ทิ้งที่ขั้นยินยอมแล้วยังมีแถว member ค้าง

ดู [[reference_local_qa_run_facts]] [[project_oc4523_line_login_cards]]
