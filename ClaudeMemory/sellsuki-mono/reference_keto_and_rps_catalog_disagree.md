---
name: reference_keto_and_rps_catalog_disagree
description: "permission ที่ Keto บังคับใช้อยู่จริง อาจไม่มีใน catalog ของ rps เลย — เช็คสองที่เสมอ ไม่งั้นสรุปผิดว่า feature ยังไม่ถูก grant"
metadata:
  node_type: memory
  type: reference
---

ยืนยันบน **dev** 2026-09-14 ตอนจะ grant permission ของคูปอง:

| permission | `rps GetPermissionByCode` | Keto tuples |
|---|---|---|
| `oc2plus.member.view` | มี | ≥5 |
| `oc2plus.news.manage` | **not found** | **≥5** |
| `oc2plus.coupon.manage` | not found | **0** |
| `oc2plus.coupon.redeem` | not found | **0** |

`oc2plus.news.manage` (OC-4356) **ถูก grant ใน Keto จริงและบังคับใช้ได้** ทั้งที่
**ไม่มีอยู่ใน catalog ของ rps เลย** ⇒ ตอนนั้นมีคน grant ด้วยการเขียน Keto tuple ตรง ๆ
ไม่ได้ลงทะเบียนผ่าน rps

**How to apply:**
- **อย่าสรุปจากที่เดียว** — `GetPermissionByCode` ตอบ not found ไม่ได้แปลว่า feature นั้น 403
  อยู่ (Keto อาจมี tuple แล้ว) และการมีใน rps ก็ไม่ได้แปลว่ามีคนถูก grant
  เช็คทั้ง `rps GetPermissionByCode` และ `GET keto-read /relation-tuples?namespace=permissions&object=<code>`
- rps บน dev-th อยู่ที่ `sellsuki-dev/sellsuki-role-and-permission-management-backend-svc`
  **ไม่ใช่** `share-dev` (ที่นั่นมีแค่ keto/kratos/hydra) · gRPC 50051 reflection เปิด
- `ListRoles` ใช้ `filter_options.owner_id` + `owner_kind` (ไม่ใช่ field `owner`)
  · dev company `a3fa1608-…` มี 6 role: Company Owner (84954, system, 65 perms),
  Manager, Marketer, Warehouse, Finance Manager, Store Admin (84958)
- `UpdateRole` เป็น replace ทั้งชุด ต้อง GetRole มาก่อนเสมอ [[reference_rps_grant_permission_to_role]]

เชื่อม [[project_oc4526_coupon_wallet]] [[project_ccs_role_presets_apply_only_at_creation]]
[[project_pointclaim_permission_missing_from_owner_preset]]

## 🔴 rps ปฏิเสธการแก้ system role — นี่คือเหตุผลที่ news.manage ไปอยู่ใน Keto ตรง ๆ

ยืนยัน 2026-09-14: `UpdateRole` บน role ที่ `isSystemRole: true` ตอบ
**`Internal: system role action not allowed`** · ทุกบริษัทที่สร้างผ่าน CCS ได้ role ชุด
Company Owner/Manager/Marketer/Warehouse/Store Admin/Finance Manager มา และ
**Company Owner เป็น system role** ⇒ เติม permission ใหม่ให้ Company Owner ผ่าน
UpdateRole ไม่ได้เลย

⇒ ที่ `oc2plus.news.manage` มี Keto tuple ทั้งที่ไม่มีใน catalog ของ rps **ไม่ใช่ความมักง่าย**
แต่เป็นเพราะทางปกติถูกปิด · อย่าไปสรุปว่าคนก่อนหน้าทำลัดโดยไม่จำเป็น

**ทางที่เหลือสำหรับ system role** (ยังไม่ได้ลอง ณ 2026-09-14):
`MigrateRoles(roles: []UpdateRoleRequest, actor)` — น่าจะเป็นเครื่องมือที่ตั้งใจไว้สำหรับ
กรณี preset เปลี่ยน แต่ชื่อ + signature กำกวมว่าเป็น "แทนที่ทั้งชุด" หรือไม่ **อย่ายิงมั่ว
บน dev/staging ที่ใช้ร่วมกัน** · หรือสร้าง role ใหม่ที่ไม่ใช่ system role แล้ว assign แทน

**ที่ทำได้จริงและยืนยันแล้ว:** role ที่ไม่ใช่ system role แก้ผ่าน UpdateRole ได้ปกติ และ
rps **propagate ลง Keto ให้เอง** (Store Admin 84958 ได้ `oc2plus.coupon.redeem`
แล้ว Keto ขึ้น tuple ทันทีโดยไม่ต้องเขียน Keto เอง)
