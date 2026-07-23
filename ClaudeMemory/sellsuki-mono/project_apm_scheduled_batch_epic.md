---
name: project-apm-scheduled-batch-epic
description: "BOLA APM scheduled send + batch digest — design decisions เคาะแล้ว, epic + 6 stories publish ลง Jira BOLA (2026-07-23)"
metadata: 
  node_type: memory
  type: project
  originSessionId: fe3c565b-95d2-4c0e-9ab7-b851612ffe9f
  modified: 2026-07-23T00:37:37.595Z
---

ฟีเจอร์ Auto Push Message: Scheduled send + Batch digest (ต่อยอดจาก [[project-bola-apm-webhook-design]]) — ออกแบบเสร็จ + สั่ง publish การ์ดลง Jira project BOLA แล้ว 2026-07-23 (1 Epic + Story A–F: A schema/queue, B webhook fork, C flush cron+batching, D FE config, E FE queue view, F QA e2e; A blocks ทุกใบ, ทุกใบ blocks F)

**Design decisions ที่เคาะแล้ว (อย่า re-litigate):**
- Cron = `bola_server --mode=worker --job=auto-push-flush` (pattern เดิม 4 jobs) **ไม่สร้าง binary ใหม่** — แต่ BOLA-250 (cron migration, Ready for QA) อาจเปลี่ยนเป็น separate binary → align ตอน implement, link เป็น relates ไม่ใช่ blocked-by
- `webhook_payload` ในตาราง `auto_push_queued_deliveries` เก็บ **TEXT** (cast `::jsonb`) ตาม convention bola
- Batching = **text เท่านั้น phase 1** (flex→carousel สูงสุด 12 bubbles = phase 2); LINE นับ quota ต่อ push request (≤5 message objects/push = 1 quota) → รวบ N payload ประหยัด N-1 quota
- Render template **ตอนรับ webhook** (ค่า ณ event time) เก็บ rendered_message ลงคิว; dynamic target batch **ต่อ user ห้ามข้าม user**
- `next_run_at` บน APM = source of truth รอบถัดไป; queue ว่าง = ไม่ส่ง แค่เลื่อนรอบ; APM disabled = skip แต่ advance
- Queue endpoints: repo methods ใน Story A, HTTP routes ใน Story E

**Jira BOLA issue types (verified):** Epic 10000, Story 10014, Task 10155, Sub-task 10156, Bug 10157. การ์ดเดิมที่เกี่ยว: BOLA-57 (APM epic แม่), BOLA-250 (cron→K8s CronJob), BOLA-270 (group target), BOLA-169 (timezone bug precedent)

**Why:** design ถูก challenge/ปรับหลายรอบก่อนเคาะ — ถ้าเปิด session ใหม่มาทำ Story ใดๆ ต้องยึด decision ชุดนี้
**How to apply:** ก่อน implement story ในชุดนี้ อ่าน memory นี้ + การ์ด Jira จริง (หา key ด้วย JQL `project=BOLA AND summary~"APM Schedule"`)
