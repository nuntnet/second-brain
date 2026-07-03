---
name: feedback-subagent-model-tiering
description: จัด model ของ sub-agent ตามความยากของงาน — อย่าใช้ model แพงสุดกับทุกตัว
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b0b4a24f-96f9-41ef-b15d-18e86ad938e1
---

User สั่ง (2026-07-03, ตอนรัน /po-team บน Fable): "agent อื่นๆ ที่ไม่ต้องใช้ fable ก็ให้ใช้ model อื่นๆ ตามความยากง่ายของงาน"

**Why:** ประหยัด cost/เวลา — งาน scan/inventory ไม่ต้องการ model ใหญ่; เก็บ model แพงไว้กับ synthesis และ adversarial verify เท่านั้น

**How to apply:** เมื่อ orchestrate ทีม sub-agent: scan/mechanical → haiku, research/cross-reference → sonnet, deep verify/hard analysis → opus, synthesis ที่ตัว lead → inherit session model. หมายเหตุ: `.claude/rules/multi-repo.md` เขียนว่าใช้ sonnet ทุกตัว — คำสั่งนี้ refine เป็น tiering ตามความยากแทน. เกี่ยวข้อง [[feedback-decisive-deep-execution]]
