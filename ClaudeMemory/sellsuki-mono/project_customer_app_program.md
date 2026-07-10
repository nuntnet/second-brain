---
name: project_customer_app_program
description: "Customer App program (แทน posh-medica): 5 epics label customer-app-program — OC-4344 foundation/4349 points/4355 CMS/4359 profile + earning ใต้ OC-2743; BFF ที่ member-api; ไม่สร้าง repo ใหม่"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

**OC2Plus Customer App & CMS program (แทน posh-medica) — published 2026-07-10** · label `customer-app-program` · P1 (pending SoT/control-tower update)

**Architecture ที่เคาะ (PO confirm 2026-07-09):** multi-tenant stack เดียว (posh migrate เป็น tenant ทีหลัง) · **member-api = Customer BFF** — browser ห้ามยิง 3rdparty-api ตรง · ไม่สร้าง repo ใหม่เลย (frontend ขยาย `oc2plus-linecrm-frontend-member`, CMS authoring ใน backoffice-api, engine = 3rdparty-api เดิม) · theme = ขยาย token model OC-4246 เป็น app-wide serve ผ่าน /v1/theme (ฆ่า pain posh ที่ต้อง redeploy ต่อแบรนด์) · **web login (non-LIFF OTP) อยู่ใน MVP** (PO ยืนยัน — ต่างจากที่ทีมแนะนำ LIFF-first) · News CMS = editor เต็ม · cache: theme/news/promo ได้, point balance/redeem confirm ห้าม

**Epics:** OC-4344 Foundation (4345 spike auth⚠️critical path, 4346 shell, 4347 BFF, 4348 web login, 4246 theme) · OC-4349 Points/Rewards (4350-4354) · OC-4355 News CMS (4356-4358) · OC-4359 Profile/Consent (4360-4361 thin wiring) · earning ใหม่ใต้ OC-2743 เดิม (4362 no-QR receipt, 4363 code entry) · 25 dependency links

**สถานะปิดงาน publish (2026-07-10): DoR 18/18 story ผ่าน** — blockers แก้หมด: OC-4362 เคาะ **reject อัตโนมัติ+ข้อความ** (แต้มพิเศษ→OC-4294) · OC-4345 timebox 3 วัน · OC-4358 cache key scope company_id · เพิ่ม **OC-4364** News scheduling worker (แตกจาก OC-4356) · harvest pointers แปะครบ 5 ใบ (4363/4335/4354/4353/4347) · **นโยบาย posh code: ไม่ fork — selective harvest logic-only** (PrimeVue markup ห้าม copy; posh ไม่มี i18n/LIFF) · caveat: OC-4246/4345 checkbox render เป็น text (MCP markdown→ADF bug — แก้ได้เฉพาะ Jira UI) · critical path = spike OC-4345 gate ทั้งโปรแกรม

**posh-medica clone อยู่ scratchpad (3 repo: customer-fe 31 routes, backoffice-fe = banner/channel/minigame/promotion, backend-go = BFF pattern)** — path จริง `sellsuki/beary/posh-medica/{backend,frontend}/...` (subgroup ซ้อน) · บทเรียนจาก posh: QR scanner ต้องมี 3-engine fallback (jsQr/jsQr3x/zxing — camera บน LINE browser), มี /r/:code short-link, /delete-account (PDPA) · minigame/marketplace = out of scope v1

เชื่อม [[reference_oc2plus_member_frontend]] [[project_loyalty_point_cluster]] [[project_oc_bola_domain_boundary]]
