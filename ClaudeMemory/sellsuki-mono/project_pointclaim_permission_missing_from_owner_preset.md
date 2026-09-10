---
name: project_pointclaim_permission_missing_from_owner_preset
description: oc2plus.pointclaim.review ขาดสองชั้น — ไม่มีใน permission_lists ของ rps และไม่มีใน preset Company Owner ของ CCS → ไม่มีบริษัทไหนเห็นเมนูคำขอแต้ม; แก้ด้วย rps 0021 + CCS _0019 (เปิด MR 2026-09-10)
metadata: 
  node_type: memory
  type: project
  originSessionId: f1e1e4b4-53e4-4459-99e1-ea7fc29faaa9
  modified: 2026-09-10T16:13:39.260Z
---

**อาการ:** เมนู "คำขอแต้ม" ไม่ขึ้นใน sidebar ของ OC2Plus backoffice เลยสักบริษัท
(`SideBar.vue` ซ่อนเมนูเมื่อ `usePermissions([PERMISSIONS.POINT_CLAIM_REVIEW])` ไม่ผ่าน)

**สาเหตุจริง — ขาด 2 ชั้น ไม่ใช่ชั้นเดียว** (แก้ครั้งแรกเข้าใจผิดว่าขาดแค่ preset):

1. **rps `permission_lists` ไม่มี row นี้เลย** — `grep -rn "pointclaim" backend/sellsuki-role-and-permission-management-backend`
   คืนค่าว่าง ทั้ง repo (ยืนยัน 2026-09-10) มันมีแค่ใน `backend/entity/access_control/permission_list.go`
   (Go constant) ซึ่งเป็นคนละที่กัน · **permission ที่ไม่มีใน catalog แปะเข้า role ไม่ได้**
   จึงเป็นไปไม่ได้ที่จะ grant แม้จะเขียน CCS migration แล้ว
2. **CCS preset ไม่มี** — `CompanyOwnerPermissions` ไม่มีมัน และไม่มี `_00NN_migrate_permission_*`
   ใบไหนที่ back-fill ให้ role ที่มีอยู่แล้ว

**ผลกระทบ:** OC-4362/4464/4512/4524/4461 merge แล้วแต่มองไม่เห็นจาก UI ทุก environment
(ซ่อนเมนูฝั่ง UI ไม่ใช่การป้องกัน — endpoint ยัง 403 ถูกต้อง แต่ก็แปลว่าเข้าไม่ได้เลย)

**การแก้ (เปิด MR แล้ว 2026-09-10):**
- rps `cmd/migration/migrations/0021_add_oc2plus_pointclaim_review_permission` — seed row
  (`member_management`, `oc2plus.company`, priority 31) · **เลข 0021 ไม่ใช่ 0019** เพราะ main อยู่ที่
  0020 ส่วน develop ค้างที่ 0018 (develop ขาด 0019 point_adjust และ 0020 sukipay ทั้งคู่) —
  gormigrate key ด้วย string id ต้องข้ามให้พ้นเลขสูงสุดของ**ทั้งสองสาย** · MR !119 (main) / !120 (develop)
- CCS `_0019_migrate_permission_oc2plus_pointclaim_review_to_CompanyOwner_and_CompanyMarketer`
  + เพิ่มใน `CompanyOwnerPermissions` และ `CompanyMarketerPermissions` · **ต้องทำทั้งสองอย่าง**:
  preset ใช้ตอนสร้าง role เท่านั้น ไม่เคย re-apply → บริษัทที่มีอยู่แล้วต้องพึ่ง migration
  · MR !327 (main) / !328 (develop)
- ให้ Marketer ด้วยตาม `_0018` (point_adjust) — ทั้งสอง role ถือ `oc2plus.point.adjust` อยู่แล้ว
  ซึ่งแรงกว่า; ให้ปรับแต้มมือได้แต่ห้าม approve claim ที่ขอแต้มนั้นมันขัดกันเอง

**ทางลัดสำหรับ local เท่านั้น:** assign role ของบริษัทอื่นข้ามมาได้ (rps `AssignRole` ไม่ตรวจว่า role
เป็นของ tenant นั้นจริง — ดู [[reference_rps_internal_endpoints_confirmed]]) เช่น role 65

เชื่อม [[project_ccs_role_presets_apply_only_at_creation]] [[reference_rps_dual_mainline]]
[[project_oc4362_claim_cluster_gaps]] [[reference_rps_internal_endpoints_confirmed]]
