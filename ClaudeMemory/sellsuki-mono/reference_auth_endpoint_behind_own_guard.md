---
name: reference_auth_endpoint_behind_own_guard
description: bug class — เอา middleware ที่ต้องการ identity ไปครอบ endpoint ที่ทำหน้าที่ "ผลิต" identity = ทั้งระบบ 401 หมด
metadata:
  type: reference
---

**bug class:** gateway (Ory Oathkeeper `bearer_token` → `check_session_url`) เรียก endpoint ของ service เองเพื่อยืนยัน credential
ถ้า middleware ที่ตรวจ identity ถูก mount ทั้ง route group รวม endpoint นั้นด้วย → endpoint ที่ *ผลิต* identity ถูกบังคับให้ *แสดง* identity ที่ยังไม่เกิด
⇒ gateway ยืนยันตัวตนใครไม่ได้เลย → ตกไป authenticator `anonymous` → **ทุก endpoint ตอบ 401 แม้ credential ถูกต้อง**

เจอจริง OC-4466 (oc2plus 3rdparty-api, กิน dev-th ไป 18 วันโดยไม่มีใครรู้) — `RequireApiKeyScope` มี `v2NoScopeRequired` ยกเว้นให้ `/auth/whoami` เฉพาะการเช็ค **scope** แต่ไม่ได้ยกเว้นการเช็ค **identity**

**วิธีจับ:**
- **ดูเวลา** — 401 ใน ~40 ms = ไม่ได้แตะ bcrypt/DB เลย แปลว่าถูกตัดก่อนถึงชั้น verify · ของจริงที่ verify แล้วปฏิเสธจะใช้เวลาระดับ 200+ ms (bcrypt)
- ยิง endpoint นั้นตรงที่ pod **สองรอบ**: รอบแรกส่งเฉพาะ credential ดิบ (แบบที่ gateway ส่งจริง) รอบสองใส่ identity header ปลอมเพิ่ม — ถ้ารอบสองผ่าน แปลว่า middleware คือปัญหา ไม่ใช่ credential

**ทำไม test ไม่จับ:** test ของ endpoint นั้นมักถูกเขียนโดยใส่ header ที่ gateway จะฉีด *หลัง* ยืนยันตัวตนสำเร็จ — mental model เดียวกับบั๊ก จึงเขียวตลอด · test ที่ถูกต้องต้องส่ง request แบบที่ gateway ส่งจริง (credential ดิบล้วน) และต้องแดงเมื่อถอด fix ออก

**กฎ:** route ที่ *establish* identity ต้องอยู่นอกด่านที่ต้องการ identity เสมอ และไม่ได้อ่อนแอลง เพราะ handler ของมัน verify credential กับ hash เองอยู่แล้ว (แข็งกว่าเช็ค "header ไม่ว่าง")

ดู [[project_oc2275_audit_actionplan]] · [[feedback_verify_as_the_user_sees_it]] · [[reference_timing_dependent_concurrency_tests]]
