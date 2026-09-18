---
name: reference_oc2plus_dev_urls_and_multicompany_grant
description: "URL ของ OC2Plus dev ไม่มี -th (crm.dev.oc2.plus) และ user เทสมี role ข้าม 3 บริษัท — grant บริษัทเดียวไม่พอ"
metadata:
  node_type: memory
  type: reference
---

## URL dev ไม่มี `-th`

- backoffice: **`crm.dev.oc2.plus`** (ไม่ใช่ `crm.dev-th.oc2.plus`)
- member app: **`member.dev.oc2.plus`** (ไม่ใช่ `member.dev-th.oc2.plus`) — ยืนยัน
  2026-09-18 ด้วย DNS: `member.dev-th.oc2.plus` **ไม่ resolve เลย** ส่วน
  `member.dev.oc2.plus` → 3.166.34.69 · ค่าที่เคยจดว่า `-th` มาจาก config ใน repo
  ซึ่งไม่ตรงของจริง ⇒ **อย่าอนุมานจาก config** เช็ค DNS หรือ ingress ก่อน
- slug ที่ใช้ได้บน dev = **`d9pesc1nm7id80npgljg`** = `code` ของ company C
  (`a3fa1608-…`) · เปิด `https://member.dev.oc2.plus/d9pesc1nm7id80npgljg/login`
  ได้หน้า "เข้าสู่ระบบสมาชิก" จริง (2026-09-18) · SPA คืน 200 ทุก path ⇒ curl
  status code **พิสูจน์ slug ไม่ได้** ต้องเปิดเบราว์เซอร์ดู

## login บน dev ทำเองไม่ได้ ถ้าไม่มี TEST_KEY

`TEST_KEY` มีอยู่บน dev (secretKeyRef ใน deployment) แต่เบราว์เซอร์ไม่ได้ฉีด
เฮดเดอร์ `X-TESTING-SECRET`/`X-Test-Secret` ให้เหมือน vite proxy บนเครื่อง ⇒
กรอกเบอร์ในหน้า login บน dev = **ขอ OTP ผ่านของจริง** เพราะ
`MESSAGING_SERVICE_BASE_URL=http://sellsuki-messaging-backend-svc.sellsuki`
(namespace `sellsuki` ไม่ใช่ mock) · อย่ายิง OTP ใส่เบอร์คนอื่นเพื่อเทส

## user เทสบน dev มี role ข้าม 3 บริษัท — grant บริษัทเดียว = ยังเจอ 403

`fad32b4d-1c84-407d-ba96-c1380114f828` (คนเดียวที่มี role ใน CRM dev) ถือ
Company Owner + Store Admin ใน **3 บริษัท**:

| company | ชื่อที่ backoffice-api คืน | ข้อมูล CRM |
|---|---|---|
| `a3fa1608-a0ea-4681-b5c7-bbbb9f47245e` | company C | **member + coupon ทั้งหมดอยู่ที่นี่** |
| `3265acf7-b1f1-4648-8425-698091863630` | company A | ไม่มี member ใน CRM |
| `7ba24401-3d7b-412b-b47d-8226cb5c9e37` | company B | ไม่มี member ใน CRM |

Keto ผูก permission กับ **role ของบริษัทนั้น ๆ** ⇒ grant ที่ company C แล้วเปิด UI
ของ company A ก็ยัง 403 · หา tenant ทั้งหมดของ user ด้วย
`ListAssignedRoles{user:{kind,id}}` (field ชื่อ `user` ไม่ใช่ `identity`) ซึ่งคืน
`assignments: {"sellsuki.company:<id>": {roleIds: [...]}}`

**ชื่อบริษัทที่โชว์บน sidebar (เช่น "มหานคร") ไม่ตรงกับชื่อที่ `GET /v1/company/{id}`
ของ backoffice-api คืน** (ได้ "company A/B/C") — ชื่อ display มาจากที่อื่น อย่าใช้ชื่อ
เป็นตัวจับคู่ ให้ใช้ company id

## endpoint ที่ FE ใช้เช็คสิทธิ์ (ไว้ debug 403)

`POST /v1/company/{company_id}/permission` body `{"permission":["<code>", …]}`
→ `{"results":[{"name","is_allow"}]}` · FE ถามเป็นรายชื่อ ไม่ได้ขอทั้งหมด
(`src/stores/permission/index.ts` + `composable/usePermissions.ts`) และเก็บผลไว้ใน
Pinia ทั้ง session ⇒ **grant ระหว่างที่เปิดหน้าอยู่ ต้อง reload ถึงจะเห็น**

ยิงตรงด้วย port-forward `svc/oc2plus-line-crm-service-backoffice-api-svc 18092:80`
แล้วใส่ header `X-User-Id` / `X-User-Kind` (prefix `/backoffice` เป็นของ gateway
ยิงตรงต้องตัดออก เหลือ `/v1/...`)

เชื่อม [[reference_keto_and_rps_catalog_disagree]] [[project_oc4526_coupon_wallet]]
