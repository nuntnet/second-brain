---
name: project-pat-phase5-bola-selfhost-patconnex
description: "การท่าเรือ (PAT) Phase 5 — BOLA self-host บน server ของ PAT + PAT Connex ยิง API เข้า BOLA · epic BOLA-382, การ์ด BOLA-383..388 (สร้าง 2026-11-10) · ข้อเท็จจริงจากโค้ดที่ใช้ตั้งการ์ด และคำตัดสินของ PO"
metadata:
  node_type: memory
  type: project
  originSessionId: 300c6962-12fa-4740-9aa3-3eb3ab997d45
  modified: 2026-11-10T16:05:17.784Z
---

**โครงการ:** PAT LINE OA Management System (Phase 5) · proposal `~/Downloads/PAT_BOLA_Proposal.pdf` (11 หน้า) · 2 OA: PATCONNEX (ลูกค้า), ทันข่าว (พนักงาน + เกษียณ) · PAT Connex = ระบบเดิม เป็น source of truth มี LIFF form · 198,000/ปี ไม่มี one-time · 85 man-day, 60 วันทำการ · "custom web" ที่ nunt พูดถึง = PAT Connex (การอ่านของฉัน ยังไม่ยืนยัน) codebase ไม่อยู่ใน workspace

**การ์ด (Jira BOLA, สร้าง 2026-11-10):** epic BOLA-382 · 383 Contact API (เมนูสลับ + tenancy + ไม่ทับข้อมูล LINE) · 384 API key ของ workspace (header `X-API-Key`, key ขึ้นต้น `bola_`) · 385 สัญญาเชื่อมต่อ + ชุดตั้งค่า 2 OA · 386 campaign หลาย OA เลือก segment ต่อ OA · 387 bug เพดานผู้รับ 10,000 · 388 ติดตั้งบน server ลูกค้า (ไม่ Ready รอ BOLA-231) · ร่างและ journey: `docs/cards/drafts/bola/PAT-phase5.md` · ledger `docs/cards/BOLA-383.md` · UX brief 384/386 ผ่าน ready check · inbox D-339..D-345

**คำตัดสินของ PO (2026-11-10):** upsert ต้องสลับเมนูเอง → ทำ · ช่องยืนยันตัวระบบต่อระบบ → ทำ · multi-OA broadcast แบบ multi select → อยากได้ · แพ็กเกจติดตั้ง = เนื้องาน · ผู้เกษียณเห็นเมนูต่าง · `personal_id` ตัดถ้าอ่อนไหว · โปรไฟล์ร่วมกันทั้ง workspace โดยตั้งใจ · tag แทนที่ได้ · "เขียนการ์ดแล้วทำต่อได้เลย"

**ข้อเท็จจริงจากโค้ด bola-backend origin/main `4eb5e44` (อ่านโค้ด, ผู้ตรวจอิสระอ่านซ้ำแล้ว, ยังไม่ได้ทำซ้ำบนระบบจริง ณ เวลาที่เขียน):**
- `UpsertContacts` ไม่เรียก rich menu re-apply · ปุ่ม URI ไม่ยิง webhook
- upsert ไม่ตรวจว่า `line_oa_id` เป็นของ workspace ผู้เรียก และ `ON CONFLICT (line_oa_id, line_user_id) DO UPDATE` เขียนทับ display_name/picture/follow_status ฯลฯ + `deleted_at = NULL` (`customer_repository/postgres_gorm.go:57`) — คาดว่าเป็นช่องข้าม tenant บน SaaS
- upsert ใช้ FlatAdminGuard (JWT 9 ชม.) · `X-System-Token` มีแต่เป็นค่าเดียวทั้งระบบ ห้ามให้ลูกค้าถือ
- Contact API Logs ผูก workspace จากแอดมินใน request
- **wizard ของ broadcast เลือกหลาย OA ได้อยู่แล้ว** (ที่ฉันบอก nunt รอบแรกว่าต้องสร้าง 2 ครั้งนั้นผิด) แต่ segment ตัวเดียวถูกใช้ทุก OA และตอนส่งดึงผู้รับจาก OA ของ segment (`auto_push_message_delivery.go:324`) · handler ไม่ตรวจ workspace ของ segment
- ดึงผู้รับด้วย `Limit: 10000` ทั้ง broadcast และ auto push message
- bola-frontend ไม่มี i18n library: ใช้ module ต่อหน้า (`lineOAI18n.ts`, `src/lib/broadcastListI18n.ts`) ภาษาตาม navigator.language

**การออกแบบที่ฉันตัดสินแทน (ใน inbox):** กฎเมนู/segment ใช้ custom field (`pat_user_type`, `pat_company_type`, `pat_company_name`, `emp_type`, `emp_department`, `emp_id`, `emp_is_retired`) ไม่ใช้ tag เพราะโปรไฟล์ร่วมกันและ tag ถูกแทนทั้งชุด · API key: super_admin, 10 key, 600/นาที

**Self-host:** BOLA-231 (Docker Compose) และ BOLA-229 (license) ยัง To Do · rgb72 รันบน EKS ของ Sellsuki ดู [[project_bola_deploy_topology]]

**How to apply:** ก่อนทำการ์ดใดในชุดนี้ให้อ่าน ledger และ `docs/cards/drafts/bola/PAT-phase5.md` · งานฝั่ง PAT Connex ยังไม่มีเจ้าของ (D-344) · ต้องถามว่า LINE Login ของ LIFF อยู่ Provider เดียวกับ 2 OA หรือไม่
