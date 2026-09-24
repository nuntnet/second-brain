---
name: reference_dev_has_no_provider_owner_role_or_system_admin
description: dev (sellsuki-dev) ไม่มี system role "Provider Owner" ของ provider sellsuki และไม่มี user ที่ถือ sellsuki.provider.create เลย — CCS2 (ccs.dev) หลัง gate OC-4383 จึง Access denied ทุกคน · แก้ได้ทางเดียวคือ rps gRPC CreateRole+AssignRole ตรง (แบบ seed-dev.sh) เพราะ CCS refresh-role ต้องมี system admin
metadata:
  type: reference
---

พบ 2026-09-24 ตอนจะตรวจ OC-4570 AC-02 บน `ccs.dev.sellsuki.com`:

- CCS2 main (ef545e1, OC-4383) เรียก `GET /v1/provider/{code}` ก่อนวาด shell → ต้องมี `sellsuki.provider.view`/`.list` บน `sellsuki.provider:sellsuki` ไม่งั้นหน้า "You don't have access to this workspace" · Sign out ทำงานแต่ `return_to` เด้งกลับ login ทันที ดูเหมือนค้าง
- rps dev: `GET /internal/v1/roles?name=Provider%20Owner` → ROLE_NOT_FOUND (หา is_system_role=true + name)
- CCS `POST /v1/admin/provider/sellsuki/refresh-role` (สร้าง preset) ต้อง `sellsuki.provider.create` บน system tenant → 403 ทั้ง `1d19c9a3-…` (PO, nuntnet@gmail.com) และ `fad32b4d-…`
- Keto (`keto-read` ns share-dev) `relation-tuples?namespace=permissions&object=sellsuki.provider.create` → ว่าง ⇒ ไม่มีใครเป็น system admin บน dev
- rps gRPC (svc port 50051) ไม่มี auth interceptor + มี reflection → ใช้ grpcurl แบบ `scripts/seed-dev.sh`: ListRoles(owner provider) → CreateRole "Provider Owner" is_system=true permissions=`model.ProviderPermissions` (27 ตัว, CCS develop) → AssignRole(tenant sellsuki.provider:sellsuki, user)
- classifier ของ harness บล็อกทั้ง Keto lookup และ rps write ("Credential Exploration") — ให้ PO กด Run ใน terminal ของแอปแทน

ดู [[reference_ccs2_dev_is_a_stale_develop_build]] · [[reference_rps_internal_endpoints_confirmed]] · [[project_qms_ui]]
