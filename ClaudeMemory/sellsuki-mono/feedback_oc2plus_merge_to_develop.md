---
name: feedback_oc2plus_merge_to_develop
description: OC2Plus MRs ทุกตัวต้อง merge to develop เท่านั้น ห้าม merge to main
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-07-20T05:12:56.491Z
---

**กฎ (user สั่ง 2026-07-20):** MR ของงาน OC2Plus **ทุกตัวต้อง target `develop` เท่านั้น ห้าม merge เข้า main**

**ครอบคลุมถึง platform service กลางด้วย** (group `sellsuki/sellsuki`) เมื่อ MR นั้นเป็นส่วนของ OC2Plus delivery — user ยืนยัน "ครอบทั้งหมด → develop" รวม role-permission, CCS, ไม่ใช่แค่ repo ที่ OC2Plus เป็นเจ้าของ

**How to apply:** ตอนสร้าง/ตรวจ MR ของงาน OC (เช่น OC-4257 QMS chain) — เช็ค target branch เป็น develop; ถ้าเผลอ target main ให้ `glab mr update <id> --target-branch develop` ทุก repo มี `origin/develop`

**Why:** main = release branch, develop = integration; OC2Plus flow รวมงานที่ develop ก่อน

QMS delivery 2026-07-20: !194 (mgmt-backend), !7 (provider-fe), !86 (role-permission), !253 (CCS feature/provider) — retarget → develop หมด · !194/!7 สะอาด, !86/!253 ต้องแก้ conflict (develop นำหน้า) · feature/provider แยก QMS commit ออกไม่ได้ (พึ่ง checkPermissionForCompanyOperation จาก PAT-2448 03b000f) เชื่อม [[project_qms_ui]]
