---
name: reference_generated_file_merge_conflict_regenerate
description: conflict ใน spec.gen.go / ไฟล์ generated ให้ merge yaml แล้ว regenerate ห้ามแก้ด้วยมือ — และห้าม rebase branch ที่ push แล้ว
metadata:
  type: reference
---

MR ที่ conflict เฉพาะไฟล์ **generated** (`spec/v1/spec.gen.go`) แต่ **source (`v1.yaml`) auto-merge ผ่าน**
คือเคสที่แก้ง่ายและคนมักแก้ผิด

```bash
git merge origin/develop          # ไม่ใช่ rebase — branch push แล้ว
# v1.yaml จะ auto-merge เอง, เหลือแค่ spec.gen.go ที่ UU
make gen-http-fiber               # regenerate จาก yaml ที่ merge แล้ว
git add -A && git commit
```

**ห้ามแก้ conflict marker ในไฟล์ generated ด้วยมือ** — ผลลัพธ์จะไม่ตรงกับ source
และรอบ generate ถัดไปจะพังหรือกลับค่า

**พิสูจน์ว่าถูกก่อน commit** — generate สองรอบต้องได้ hash เดียวกัน:
```bash
H1=$(shasum <file>|cut -d' ' -f1); make gen-http-fiber; H2=$(shasum <file>|cut -d' ' -f1)
[ "$H1" = "$H2" ] && echo deterministic
```
แล้วเช็คว่า endpoint ของ **ทั้งสองฝั่ง** ยังอยู่ (ของ develop ที่เพิ่งเข้ามา + ของ branch เรา)

**merge ไม่ใช่ rebase** เมื่อ branch push ไปแล้ว — rebase = ต้อง force push ซึ่ง
`.claude/CLAUDE.md` ห้ามเด็ดขาด ดู [[feedback_ff_only_force_push_ok]]

ใช้จริง 2026-09-12 กับ member-api MR !112 (OC-3559) ที่ค้าง conflict อยู่
