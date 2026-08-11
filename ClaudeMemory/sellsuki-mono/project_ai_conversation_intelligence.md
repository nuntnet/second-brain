---
name: project_ai_conversation_intelligence
description: AI chat platform §5.13 — Case/Checkpoint/Memory-lanes/Mismatch design เคาะ 2026-08-11 + รายการการ์ดที่ต้องแก้ตาม
metadata: 
  node_type: memory
  type: project
  originSessionId: 1e84c3ff-15a2-43f1-989e-32ce9d630f3d
  modified: 2026-08-11T16:53:15.023Z
---

**§5.13 ของ `docs/ai-chat-assistant-platform-plan.md` (ACTIVE, user เคาะครบ 4 ข้อ 2026-08-11)** — เกิดจากคำถาม "AI ต้องสรุปบทสนทนา รู้ว่าคุยถึงไหน และรู้ตัวเมื่อ serve wrong context":

1. **Case = หน่วยของ "เรื่อง"** — `Conversation ⊃ Session ⊃ Case`; case มี `subject` (self | third_party เช่นสามี) · **stage ผูก `case_id` ไม่ใช่ contact×workspace** (แก้ §5.7 R1 + AI-47 Rule 4 ที่ `src/entity/lead/lead.go` ใช้ `ContactID`) · fact scope ที่สาม `case` + `subject_ref` (แก้ R3 ที่มีแค่ workspace|company) · attribution ladder ถูก→แพง: เปิดใบเดียว → quick reply → NER → embedding (reuse Tier1) → **ถามลูกค้า** ห้าม LLM เดาเงียบ
2. **Checkpoint ≠ Stage** — Stage=ภาษาธุรกิจ, Checkpoint=`(facts × playbook) → current_step + missing_slots + next_action` เป็น **pure function คำนวณใหม่ทุกครั้ง** ไม่เก็บ mutable state; playbook = config ต่อ workspace ใน CCS namespace; **agent ไม่ตัดสิน step เอง**
3. **Memory lanes 4 ชั้น** (แทน two-tier §5.6.4): L1 verbatim · L2 narrative ต่อ **case** · L3 typed facts · L4 document memory ต่อบุคคล (แยกขาดจาก workspace KB) — **routing เป็นโค้ด deterministic ไม่ให้ LLM ตัดสิน**; rule/checkpoint ห้ามอ่าน narrative; unmapped fact → คิวให้ admin เพิ่ม field
4. **Context Mismatch Detector** — signal: correction pattern / ถามซ้ำ / **case-attribution flip** (ไม่ใช่ "ลูกค้าด่า"); act ทันที (ยกเลิกการเดา case → re-ground → ส่งต่อคน → `context_mismatch` event → golden set); fact ที่ถูกปฏิเสธ = `disputed` ไม่ลบ

**ต้องแก้การ์ด (ยังไม่เสร็จ ณ 2026-08-11):** AI-16 (scope case+subject_ref+disputed) · AI-47 (stage key=case_id) · AI-45 (summary ต่อ case) · AI-54 (assembler block Case+Checkpoint ~150 tok) · AI-55 (อ่าน missing_slots ห้ามอ่าน narrative) · AI-52 (output 2 ก้อน + unmapped queue) · AI-95 (case_id/checkpoint ในทุก event — backfill ไม่ได้)
**การ์ดใหม่ 5 ใบ:** Case aggregate+attribution (AI-3) · Playbook/Checkpoint engine (AI-3) · Playbook editor (AI-9) · Context Mismatch (AI-3) · Per-customer document memory (AI-5)

Artifact diagram = `docs/ai-platform-architecture.md` → publish ที่ artifact 4972126d-7be7-4f0f-8c0a-5812e23568de (ก่อนหน้านี้ artifact นี้ไม่มี source ในรีโป) · ดู [[reference_ai_board_stale_cards]] · [[reference_ai_platform_architecture_artifact]]
