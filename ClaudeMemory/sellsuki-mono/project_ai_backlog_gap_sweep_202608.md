---
name: ai-backlog-gap-sweep-202608
description: "AI board gap sweep 2026-08-14 — สร้าง AI-142–150 ปิดช่อง §5.13/§5.11.1; ค้าง: AI-150 E8-vs-CCS3 decision + E12 อีก ~4 ใบ"
metadata: 
  node_type: memory
  type: project
  originSessionId: 22bc8e4d-cfe5-415c-8c23-d4d838fba867
  modified: 2026-08-14T14:13:58.790Z
---

Gap sweep บอร์ด AI (2026-08-14) ตอบคำถาม user "การ์ดขาดเยอะมั้ย โดยเฉพาะ admin UI":

- บอร์ดมี 141 ใบ ครอบเกือบหมด — E8 admin มี 15 ใบเดิม; §5.13.5 edits (AI-16/45/47/52/54/55/95) **ถูก apply แล้วทุกใบ** (verify ด้วย count-mode text query ตาม [[jira-mcp-search-quirks]])
- ช่องว่างจริงที่ปิดแล้ว: **AI-147** Case&Checkpoint panel ใน inbox · **AI-148** Unmapped Fact Queue · **AI-149** Quota indicator/hard-cap banner (ห้ามแก้ AI-101 เพราะ In Review — โค้ดมี indicator เป็น mock อยู่ที่ apps/admin/src/hooks/useQuotaStatus.ts) · **AI-150** Workspace Members & Roles · E12 (AI-131 เดิม 0 children): **AI-142** web chat adapter, **AI-143** Google OIDC bridge, **AI-144** reasoning-trace UI, **AI-145** Discord spike, **AI-146** template catalog+wizard

**Why:** กันทำ gap analysis ซ้ำ และกันสร้างการ์ดซ้ำรอบหน้า

**How to apply:** งานค้างถ้า user ถามต่อ — (1) AI-150 ต้องเคาะ members UI อยู่ E8 หรือ CCS3 ก่อนเข้า sprint (2) E12 ยังขาด ~4 ใบที่แผน list: HR template v1 (hr_request), staff-as-asker, ย้าย KB + ปลดระวาง channel-gateway, verify guardrails (3) AI-131 open decisions ข้อ 1/2/5 ยังไม่เคาะ
