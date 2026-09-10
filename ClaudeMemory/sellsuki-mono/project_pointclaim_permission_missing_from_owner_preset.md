---
name: project_pointclaim_permission_missing_from_owner_preset
description: "oc2plus.pointclaim.review มีใน permission catalog และโค้ดเช็คจริง แต่ไม่เคยถูกใส่ใน preset Company Owner ของ CCS → ไม่มีบริษัทไหนเห็นเมนูคำขอแต้ม จนกว่าจะเขียน migration _0018 (พบ 2026-09-10)"
metadata:
  type: project
---

**อาการ:** เมนู "คำขอแต้ม" ไม่ขึ้นใน sidebar ของ OC2Plus backoffice สำหรับบริษัทที่สร้างใหม่
(`SideBar.vue` ซ่อนเมนูเมื่อ `usePermissions([PERMISSIONS.POINT_CLAIM_REVIEW])` ไม่ผ่าน)

**สาเหตุจริง:** `oc2plus.pointclaim.review` มีอยู่ใน `entity/access_control/permission_list.go` และ
backoffice-api เช็คมันจริงทุก endpoint ของคิวรีวิว แต่ **ไม่เคยมี migration ที่เอามันเข้า preset role**
CCS มีชุด `cmd/migrate_roles_permissions/migrations/_00NN_migrate_permission_<perm>_to_<Roles>/`
ซึ่งเป็นทางเดียวที่ permission ใหม่จะเข้าไปอยู่ใน Company Owner — ล่าสุดมีถึง `_0017` และไม่มีใบไหน
เกี่ยวกับ pointclaim ยืนยันจาก rps: Company Owner ของบริษัทใหม่มี 64 permission มี oc2plus 12 ตัว
(code/campaign/member.view/reward/rewardshipment/point/broadcast/richmenu/user/bola.contact/theme/apikey)
**ไม่มี pointclaim.review**

**ผลกระทบ:** ทุกบริษัทบน staging/production จะไม่มีใครเห็นเมนูคำขอแต้ม แม้เป็น owner — ฟีเจอร์ทั้งชุด
OC-4362/4464/4512/4524 มองไม่เห็นจาก UI ทั้งที่โค้ด merge แล้ว (ซ่อนเมนูฝั่ง UI ไม่ใช่การป้องกัน
endpoint ยัง 403 ถูกต้อง แต่ก็แปลว่าเข้าไม่ได้เลย)

**วิธีแก้จริง:** เขียน migration `_0018_migrate_permission_oc2plus_pointclaim_review_to_CompanyOwner`
ใน CCS ก๊อป pattern จาก `_0015_..._oc2plus_theme_manage_to_CompanyOwner` (TargetRoles
`usecase_model.RoleOwner`, TargetPermissions `access_control.PermissionOc2PlusPointclaimReview`)
แล้วลงทะเบียนใน `migrations.go` · ต้องคุยกับทีม CCS ก่อนเพราะเป็นการเพิ่มสิทธิ์ให้ทุกบริษัทใน production

**ทางลัดสำหรับ local เท่านั้น:** assign role ของบริษัทอื่นข้ามมาได้ (rps `AssignRole` ไม่ตรวจว่า role
เป็นของ tenant นั้นจริง — ดู [[reference_rps_internal_endpoints_confirmed]]) เช่น
`grpcurl -plaintext -d '{"role_id":65,"tenant":{"kind":"sellsuki.company","id":"<company>"},...}'
localhost:9998 role_and_permission.RoleAndPermissionService.AssignRole` — ใช้ปลดล็อกเครื่องตัวเองได้
แต่ไม่ใช่การแก้จริง

เชื่อม [[project_ccs_role_presets_apply_only_at_creation]] [[project_oc4362_claim_cluster_gaps]] [[reference_rps_internal_endpoints_confirmed]]
