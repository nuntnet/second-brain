---
name: reference_rps_and_management_backend_develop_main_diverged
description: "2026-10-20 — ไม่ใช่แค่ OC2Plus: rps (project 652) develop/main เลข migration แยกสาย (develop 0024/0030/0031/0034_* vs main 0032/0033) และ management-backend (project 242, path sellsuki/patona/management-backend) ต่างกัน 52/61 commit ชน 25 ไฟล์ใน QMS · branch ที่ตัดจาก main เปิด MR เข้า develop ไม่ได้ → ใช้ twin branch (<branch>-develop = origin/develop + cherry-pick/merge) เข้า develop และเก็บ branch สะอาดไว้เข้า main"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-20T14:41:16.544Z
---

พบ 2026-10-20 ตอนเปิด MR ตาม git flow (ตัดจาก main → develop → main):

- **rps** `sellsuki-role-and-permission-management-backend` (project 652): `develop` ลงทะเบียน migration OC2Plus เป็น `0024_add_oc2plus_membertier`, `0030_add_oc2plus_member_field`, `0031_add_user_update`, `0034_add_sellsuki_stocksystem_cancel` ส่วน `main` มีงานเดียวกันเป็น `0032/0033` → `migrations.go` ชนทุกครั้งที่เพิ่ม migration ใหม่ · gormigrate ใช้ id เต็ม (string) เลขซ้ำกันสองตัวไม่ชนใน DB และ `TestMigrationsAreRegisteredInOrder` เช็กแค่ non-decreasing
- **management-backend** (project 242, GitLab path `sellsuki/patona/management-backend` — ไม่ใช่ `sellsuki/sellsuki/backend/...`): main..develop 52 commit, develop..main 61 commit, ชน 25 ไฟล์ในโซน QMS (proto, use_case/qms.go, spec) · repo นี้มี `release/vX.Y.Z` branch ของทีม Patona (pan.nit) ด้วย
- `glab`/REST: `gl.sh` ส่ง `"$@"` ตรงไป curl → ต้องใช้ `-H 'Content-Type: application/json' -d @file` ไม่ใช่ส่ง JSON เป็น positional (ครั้งแรก merge !148 ผ่านโดยไม่มี body = ใช้ค่า default ของ MR)

**How to apply:** ก่อนเปิด MR เข้า develop ของ repo ที่มี develop ให้ fetch ทั้งสองแล้ว `git merge --no-commit origin/develop` ดู conflict ก่อน · ถ้าชน: สร้าง `<branch>-develop` จาก `origin/develop` แล้ว cherry-pick commit ของ feature (ชนน้อยกว่า merge เยอะ — management-backend ชนแค่ `spec.gen.go` ที่ regenerate ได้) · pb.go ที่ merge แบบ text ต้องพิสูจน์ว่า descriptor init ได้ (เทสชั่วคราวอ่าน `File_..._proto.Messages().ByName(...)`) · เขียนใน MR + ledger ว่าเป็น twin และ branch สะอาดจะเข้า main ทีหลัง · ตัวอย่างเดิม: [[reference_oc2plus_backoffice_develop_main_diverged_2026_10_11]] · การแก้ที่ต้นเหตุ (รวมสาย) เป็นการ์ดแยก ไม่ทำกลางคืน
