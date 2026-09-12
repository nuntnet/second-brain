---
name: reference_rps_grant_permission_to_role
description: วิธี grant permission ใหม่ให้ role ในเครื่อง — Keto ผูกกับ role ไม่ใช่ user และ UpdateRole เป็น replace ทั้งชุด
metadata:
  type: reference
---

**Keto ผูก permission เข้า role ไม่ใช่ user ตรง ๆ** รูป tuple:

```
namespace              = permissions
object                 = <uuid ของ permission string>   (map ใน keto_uuid_mappings)
relation               = sellsuki.company:<companyId>
subject_set_namespace  = roles
subject_set_object     = <uuid ของ "sellsuki.role:<roleId>">
```

`object` เก็บเป็น **uuid** ไม่ใช่สตริง ต้อง join `keto_uuid_mappings` ถึงจะอ่านออก:

```sql
select mo.string_representation, mss.string_representation as granted_to
from keto_relation_tuples t
join keto_uuid_mappings mo on mo.id=t.object
left join keto_uuid_mappings mss on mss.id=t.subject_set_object
where t.namespace='permissions'
  and t.relation='sellsuki.company:<companyId>';
```

## วิธี grant (ผ่าน API ห้ามยัด SQL)

`grpcurl` มีในเครื่อง + reflection เปิดอยู่ที่ **:9998**

🔴 **`UpdateRole` เป็น replace ทั้งชุด ไม่ใช่ append** — ต้อง `GetRole` มาก่อนแล้วต่อท้าย
ไม่งั้นสิทธิ์เดิมหายหมด

```bash
grpcurl -plaintext -d '{"role_id": 65}' localhost:9998 \
  role_and_permission.RoleAndPermissionService/GetRole
# แล้วส่งกลับทั้ง list + ตัวใหม่ ผ่าน UpdateRole (ต้องมี actor + owner ด้วย)
```

role ของ local dev: **65 = "OC2Plus Company Owner (local dev)"** owner =
company `11111111-1111-4111-8111-111111111111`

## กับดักตอนเช็คผล

`POST /v1/company/{id}/permission` ของ backoffice-api ใช้ field **`permission`
(เอกพจน์)** ไม่ใช่ `permissions` · ส่งชื่อผิด → list ว่าง → rps ตอบ
`InvalidArgument: permission invalid` ซึ่ง**อ่านเหมือน code ไม่ถูกต้อง** ทั้งที่เป็น
แค่ field name ผิด

ดู [[reference_keto_staging_permission_lookup]] · [[reference_rps_internal_endpoints_confirmed]]
