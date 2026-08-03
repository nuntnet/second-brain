---
name: project_oc2plus_primary_invariant_pattern
description: "OC2Plus config entities share a \"single active primary + seed default + delete-backfill\" model"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-03T04:50:36.053Z
---

ผู้ใช้ใช้ **โมเดลเดียวกันซ้ำ** กับ config entity หลายตัวใน OC2Plus/CCS3 — เวลาออกแบบ entity ใหม่ที่มี "ตัวที่ใช้งานจริง" ให้เสนอโมเดลนี้เป็น default:

- **Seed default:** ระบบสร้าง 1 อันให้ทุก company อัตโนมัติ → ใช้งานได้ทันทีโดยไม่ต้องตั้งเอง
- **Single active primary (invariant):** ต้องมี primary/active 1 เดียวเสมอ (ไม่มี 0, ไม่มี >1) — บังคับที่ DB ด้วย partial unique index `UNIQUE(company_id) WHERE is_primary`
- **Set-primary = atomic** (ปลดอันเก่า + ตั้งอันใหม่ ใน transaction เดียว)
- **Delete rules:** non-primary ลบตรง · ลบ primary ต้อง backfill (fallback) ไปอีกอันก่อน · **ห้ามลบตัวสุดท้าย / ห้ามเหลือ 0 primary** (409)

นำไปใช้แล้ว (2026-08-03):
- **OC-4246** Customer App Theme — multi-theme/company, primary เดียว, seed, delete-backfill
- **OC-3896** Thaibulk OTP messaging config — 1 provider=1 connection, ลบ primary ที่ใช้อยู่ไม่ได้ (ต้องสลับก่อน), **แก้ของผิดใช้ Edit in-place ไม่ใช่ลบ-สร้างใหม่** (Edit = ซ่อม secret key); future multi-provider = primary/fallback
- คล้าย apikey primary ใน [[project_oc2plus_3rdparty_apikey_gap]]

ดู [[feedback_no_scope_change_in_sprint]] — การ์ด To Do แก้ model ได้; In Progress ห้ามแก้ scope. OC-3896 เป็นการ์ด Design ที่มี wireframe ~25 รูปฝัง → ตอบผ่าน comment ห้าม editJiraIssue markdown (ดู [[reference_jira_editissue_adf_breakage]]).
