---
name: oc2plus-dev-member-api-host-needs-origin
description: ยิง member-api บน dev ตรงๆ — host คือ api.member.dev-th.oc2.plus และต้องใส่ header Origin ไม่งั้น ingress ตอบ 400 เฉยๆ · member.dev.oc2.plus/v1/* ได้ HTML ของ SPA
metadata:
  node_type: memory
  type: reference
  originSessionId: 3c3cf3b4-56bc-4d5b-8f4a-c3736c29bab0
  modified: 2026-09-30T02:38:23.253Z
---

ตรวจ member-api บน dev โดยไม่มี kubectl context ของ dev (2026-09-30 มีแค่ staging-th + production):

- API host = `https://api.member.dev-th.oc2.plus` (frontend = `https://member.dev.oc2.plus/<slug>/login`)
- `member.dev.oc2.plus/v1/...` ตอบ 200 เป็น HTML ของ SPA — ไม่ใช่ API อย่าอ่านเป็น "endpoint มีอยู่"
- ไม่ใส่ `Origin: https://member.dev.oc2.plus` → envoy ตอบ `400 Bad Request` ข้อความเปล่า ดูเหมือน endpoint พัง แต่ไม่ใช่. ใส่แล้วได้ JSON error ปกติ
- ตัวอย่าง: `curl -X POST $API/v1/auth/session/renew -H 'Origin: https://member.dev.oc2.plus' -H 'Content-Length: 0'` → 401 `device_trust_invalid` = route + table `device_trust` อยู่ (ถ้า table หายจะเป็น 500)

**Why:** ตอนแรกอ่าน 400 ว่า endpoint พัง เกือบสรุปผิด. **How to apply:** smoke-test dev ผ่าน host + Origin นี้ก่อนขอ kubectl/port-forward. ใช้กับ [[project_oc4445_remember_device]] ถ้ามีไฟล์นั้น — ledger จริงอยู่ที่ docs/cards/OC-4445.md
