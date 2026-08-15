---
name: ai-backlog-gap-sweep-202608
description: "AI board gap sweep 2026-08-14 — สร้าง AI-142–150 ปิดช่อง §5.13/§5.11.1; ค้าง: AI-150 E8-vs-CCS3 decision + E12 อีก ~4 ใบ"
metadata: 
  node_type: memory
  type: project
  originSessionId: 22bc8e4d-cfe5-415c-8c23-d4d838fba867
  modified: 2026-08-15T01:53:40.354Z
---

Gap sweep บอร์ด AI (2026-08-14) ตอบคำถาม user "การ์ดขาดเยอะมั้ย โดยเฉพาะ admin UI":

- บอร์ดมี 141 ใบ ครอบเกือบหมด — E8 admin มี 15 ใบเดิม; §5.13.5 edits (AI-16/45/47/52/54/55/95) **ถูก apply แล้วทุกใบ** (verify ด้วย count-mode text query ตาม [[jira-mcp-search-quirks]])
- ช่องว่างจริงที่ปิดแล้ว: **AI-147** Case&Checkpoint panel ใน inbox · **AI-148** Unmapped Fact Queue · **AI-149** Quota indicator/hard-cap banner (ห้ามแก้ AI-101 เพราะ In Review — โค้ดมี indicator เป็น mock อยู่ที่ apps/admin/src/hooks/useQuotaStatus.ts) · **AI-150** Workspace Members & Roles · E12 (AI-131 เดิม 0 children): **AI-142** web chat adapter, **AI-143** Google OIDC bridge, **AI-144** reasoning-trace UI, **AI-145** Discord spike, **AI-146** template catalog+wizard

**Why:** กันทำ gap analysis ซ้ำ และกันสร้างการ์ดซ้ำรอบหน้า

รอบสอง (2026-08-15 หลัง /po-review): DoR review พบ owner-gap → เปิดชุด B: **AI-151** spike ย้าย internal assistant · **AI-152** HR template v1 · **AI-153** unmapped-fact queue store (E2) · **AI-154** answer-trace persist (E2, ควรเข้า pilot เพราะ backfill ไม่ได้) · **AI-155** quota status read model+event+audit (E5) · **AI-156** case-switch UI (E8, Story เต็ม) · **AI-158** workspace-scoped invite (CCS; AI-157 ถูก session อื่นจอง) — และแก้ชุด A ครบ 10 ใบ (SP field = `customfield_10016`; AI-144/146 เปลี่ยน Task→Story สำเร็จ)

**How to apply:** งานค้างถ้า user ถามต่อ — (1) decisions ที่บันทึกในการ์ดแล้วรอคนเคาะ: members UI E8-vs-CCS3 (AI-150), SSE-vs-WS ของ Inbox (AI-149), จุด terminate WS (AI-142), AI-131 ข้อ 1/2/4/5, เจ้าของ CASE_SENSITIVE_FACT_VIEWED / bulk-unassign / ai_quota_exhausted marking (2) E12 ยังขาด: staff-as-asker, ย้าย KB + ปลดระวาง channel-gateway, verify guardrails (3) frontend ที่ backend ครบพร้อมทำ: AI-138+AI-139, AI-106, AI-147 ครึ่งใบ (case routes มี, checkpoint API ยังไม่มี)
