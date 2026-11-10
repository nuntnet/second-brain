---
name: project-pat-phase5-bola-selfhost-patconnex
description: การท่าเรือ (PAT) Phase 5 — BOLA self-host บน server ของ PAT + PAT Connex (custom web เดิม) ยิง API เข้า BOLA · ช่องว่างระหว่าง proposal กับโค้ด BOLA ที่ตรวจเมื่อ 2026-11-10
metadata:
  node_type: memory
  type: project
  originSessionId: 300c6962-12fa-4740-9aa3-3eb3ab997d45
  modified: 2026-11-10T15:09:30.463Z
---

**โครงการ:** PAT LINE OA Management System (Phase 5) · proposal `~/Downloads/PAT_BOLA_Proposal.pdf` (11 หน้า, WizeMoves × Sellsuki) · 2 OA: PATCONNEX (ลูกค้า) และ ทันข่าว (พนักงาน + เกษียณ) · PAT Connex = ระบบเดิม เป็น source of truth และมี LIFF form อยู่แล้ว · ราคา 198,000/ปี ไม่มี one-time · 85 man-day, 60 วันทำการ

**"custom web" ที่ nunt พูดถึง = PAT Connex** (การอ่านของฉัน ยังไม่ได้ยืนยัน) · codebase ไม่อยู่ใน workspace นี้

**สิ่งที่ proposal ยืนยันว่า "BOLA built-in ไม่ต้องพัฒนา" แต่โค้ด origin/main v1.0.32 ไม่เป็นแบบนั้น (อ่านโค้ด ยังไม่ได้รันจริง):**
- `POST /v1/contacts/upsert` ไม่เรียก rich menu re-apply เลย · จุดที่เรียกมีแค่ follow/message webhook, UpdateFollower (แอดมินแก้ในจอ), link ของ oc2plus, และ `POST /v1/rich-menu-assignments/apply` · ปุ่ม URI บน rich menu ไม่ยิง webhook → เมนูไม่สลับหลัง login
- upsert ใช้ FlatAdminGuard = ต้องมี admin JWT (local_jwt, อายุ 9 ชม.) · ไม่มี API key · machine route `PATCH /v1/system/workspaces/:id/contacts/custom-fields` รับเฉพาะ key สงวนของ OC2Plus
- payload ใน proposal ใส่ LINE uid ใน `custom_fields.uid` · BOLA ต้องการ `line_uid` + `line_oa_id` ระดับ item ไม่งั้นไม่ผูก follower
- upsert เป็น async (202 + job_id)
- follower segment และ broadcast ผูก OA เดียว → "ส่งทั้ง 2 OA" = 2 broadcast

**Self-host บนเครื่องลูกค้ายังไม่มีแพ็กเกจ:** BOLA-231 (Docker Compose + quick-start) และ BOLA-229 (license key) ยัง To Do · rgb72 ที่เรียก self-host จริง ๆ รันบน EKS ของ Sellsuki ผ่าน chart ของ SRE ดู [[project_bola_deploy_topology]]

**การ์ดที่มาจาก PAT Phase 5 อยู่แล้ว:** BOLA-260, 261 (Ready for QA), 262 (To Do), 348, 359, 360 — เป็นงาน core ของ broadcast ไม่ใช่งาน integration · ยังไม่พบการ์ดของงาน integration เอง (ค้น Jira ด้วยคำไทยไม่ครบ)

**How to apply:** ก่อนเขียนการ์ดหรือรับปาก timeline ให้พิสูจน์ flow login → เมนูสลับ บน local ก่อน และถามว่า userId ของ LIFF กับ 2 OA อยู่ LINE Provider เดียวกันหรือไม่
