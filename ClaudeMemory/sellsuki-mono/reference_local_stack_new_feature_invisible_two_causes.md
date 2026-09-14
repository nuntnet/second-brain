---
name: reference_local_stack_new_feature_invisible_two_causes
description: ฟีเจอร์ใหม่ OC2Plus ไม่โผล่บน local ทั้งที่โค้ดอัปเดตแล้ว — เมนูซ่อนเพราะ role ขาด permission (เงียบสนิท) และ list 500 เพราะ internal API key ไม่ได้ตั้ง (fail-closed)
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-14T14:44:11.659Z
---

2026-09-14 เมนูคูปอง/ข่าวสาร/ระดับสมาชิก/API key ไม่ขึ้นบน
`oc2plus.sellsuki.local` ทั้งที่ทุก repo อยู่บน develop ล่าสุด — **สองสาเหตุซ้อนกัน
คนละชั้น** แก้ชั้นแรกแล้วจะเจอชั้นสอง

## ชั้น 1 — เมนูหาย เพราะ role ขาด permission (ไม่มีสัญญาณใด ๆ)

backoffice FE เกทเมนูด้วย `...(canManageX.value ? [item] : [])` ถ้า role ไม่มี
permission **เมนูไม่ปรากฏเฉย ๆ** ไม่ใช่ขึ้นแล้วกดไม่ได้ — ไม่มี error ไม่มี 403
ไม่มีอะไรใน log อ่านเหมือน "ฟีเจอร์ยังไม่ถูก merge" เป๊ะ

สาเหตุราก: `scripts/seed-dev.sh` grant permission ไม่ครบตามที่ FE ใช้ และ
**preset ใช้ตอนสร้าง role เท่านั้น** permission ที่เพิ่มทีหลังไม่ไหลเข้า role เดิม
(ดู [[project_ccs_role_presets_apply_only_at_creation]])

เช็ค drift:
```
grep -o "oc2plus\.[a-z.]*" frontend/oc2plus-linecrm-frontend-backoffice/src/utils/permissions.ts | sort -u
```
เทียบกับ list ใน seed-dev.sh · แก้แล้ว 2026-09-14 (เติม 7 ตัว) + ใส่คอมเมนต์เตือนไว้

**เช็คสิทธิ์จริงจาก API:** `POST /backoffice/v1/company/{id}/permission` ใช้ field
**`permission` เอกพจน์** — ส่ง `permissions` จะได้ `InvalidArgument: permission
invalid` ซึ่งอ่านเหมือนโค้ด permission ผิด ทั้งที่เป็นชื่อ field ผิด

**`ListPermissions` ของ rps ในเครื่อง panic ทุก payload** (`nil pointer
dereference`) เช็ค catalog ไม่ได้ ต้องอ้อมดูผ่าน `GetRole` · role local dev คือ
id **65** owner = company `11111111-1111-4111-8111-111111111111`

## ชั้น 2 — หน้าขึ้นแล้วแต่ list 500: internal API key

**backoffice-api ไม่ได้ถือข้อมูลคูปอง** มันพร็อกซีไป 3rdparty-api ที่
`/internal/v1/company/{id}/coupon` (`coupon_repository/http.go`) ต้องมี:
- backoffice-api: `THIRDPARTY_INTERNAL_BASE_URL=http://localhost:8103` +
  `THIRDPARTY_INTERNAL_API_KEY`
- 3rdparty-api: `INTERNAL_API_KEY` ค่าเดียวกัน

⚠️ middleware เป็น **fail-closed**: `secret == ""` → ปฏิเสธทุกคำขอ 401 ไม่ใช่
"ไม่ตั้ง = เปิดโล่ง" (`require_internal_api_key.go:27`) · อาการฝั่ง FE คือ 500
`unexpected_error` ไม่ใช่ 401 เพราะ base URL ว่างทำให้ยิงไม่ออกตั้งแต่แรก

แก้ที่ `scripts/setup.sh` แล้ว (ตัวแปร `OC2PLUS_INTERNAL_API_KEY` ค่า default
`local-dev-internal-key`) พร้อม helper **`ensure_env_key`** ที่เติมคีย์ให้ `.env`
ที่มีอยู่แล้ว — เพราะ `write_env` ข้ามไฟล์ที่มีอยู่ ตัวแปรใหม่ใน template จึงไม่มี
วันไปถึงคนที่ setup ไปก่อนหน้า (ดู [[feedback_fix_must_reach_everyone]])

ทั้งสองอย่างอยู่ใน monorepo MR !27

## ชั้น 3 (2026-09-14, เย็น) — API **ค้าง** เพราะ `go run` ไม่ rebuild

อาการ: ธีมระดับสมาชิก (OC-4553) ไม่ขึ้นบน member app เลย ทั้งที่ FE เป็น
`origin/develop` เป๊ะและสะอาด · ไล่ FE อยู่นานทั้งที่ FE ไม่ผิด

ตัวชี้ขาดคือ **ดูที่ payload ไม่ใช่ที่จอ**: `/v1/me/tier` ไม่มี field `theme`
**เลยสักตัว** (ไม่ใช่ `""` แต่ไม่มี key) → แปลว่า mapper ฝั่ง server ไม่รู้จัก
field นี้ = binary เก่า ไม่ใช่ data ว่าง

`Procfile.oc4344` รัน `go run ./cmd/generics_server` เปล่า ๆ ไม่มี air —
**คอมไพล์ครั้งเดียวตอนสตาร์ท แล้วไม่ rebuild อีกเลยตลอดชีพ** binary อายุ 6 ชม.
เสิร์ฟอยู่ โดยไม่มีอะไรบนจอหรือใน log บอก

ตรวจอายุ binary ตรง ๆ:
```
lsof -a -p <pid> -d txt -Fn | grep '^n' | sed 's/^n//' | xargs ls -l
ps -o pid,lstart -p <pid>
```
แล้วเทียบกับเวลา commit ที่คาดว่าควรจะมีผล

**แก้ถาวรแล้ว** — `Procfile.oc4344` ใช้ `scripts/svc-start.sh` + air เหมือน
Procfile หลัก (commit `8ad6abd`) · sweep แล้ว Procfile อื่นไม่มีตัวไหนเหลือ
`go run` เปล่า ๆ

**How to apply:** ฟีเจอร์ backend หายทั้งก้อนบน local → เปิด payload ดูก่อนว่า
field มาไหม ถ้า field หายทั้ง key ให้สงสัย binary ค้างก่อนสงสัย FE · และ
`overmind status` บอกแค่ "running" ไม่ได้บอกว่ารันโค้ดของเมื่อไร

เกี่ยวข้อง: [[project_oc3559_member_tier_state]] · [[project_overmind_restart_quirk]]
