---
name: background-agent-resume-patterns
description: "Subagent ตายจาก spend limit/529 resume ต่อได้ด้วย SendMessage; po-lead ชอบจบเทิร์นกลางทาง ต้องสั่ง 'ปิดให้จบในเทิร์นเดียว'; 529 ติดกัน → พัก 10 นาทีด้วย background sleep"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 22bc8e4d-cfe5-415c-8c23-d4d838fba867
  modified: 2026-08-18T17:16:01.738Z
---

บทเรียนการรัน multi-agent pipeline ยาวๆ (พบ 2026-08-18 ระหว่างงาน PO บอร์ด AI):

1. **Agent ที่ตายจาก org spend limit หรือ 529 Overloaded ไม่ได้เสียงานทิ้ง** — transcript คงอยู่ ใช้ `SendMessage` ไปที่ agentId เดิม = resume ต่อจากจุดที่ค้างพร้อม context ครบ (ใช้ได้แม้ status = failed) — ห้าม spawn ตัวใหม่ทำซ้ำ
2. **po-lead มีนิสัยจบเทิร์นเองกลางทาง** ("กำลังรอผลจากทีม") ทั้งที่ลูกจบแล้ว — ทุกครั้งที่ resume ให้สั่งชัดว่า "อย่าหยุดรอ ถ้าผลไม่ครบทำเองต่อ ปิดให้จบในเทิร์นเดียว" และอาจต้อง nudge 3-5 รอบต่อ pipeline
3. **529 ติดกันหลายรอบ = อย่า retry รัว** — ตั้ง `sleep 600` แบบ run_in_background (foreground sleep ถูก block) แล้วให้ notification ปลุกมา resume
4. Jira MCP collision ระหว่าง agent หลายตัว: ห้ามให้ main + subagent ยิง Jira พร้อมกัน (เคยได้ response ของ call อื่นสลับใบ) — ทำ Jira write ทีละผู้เล่นเสมอ ดู [[jira-mcp-search-quirks]]

**How to apply:** งาน /po-team หรือ batch Jira ใหญ่ๆ — คาดว่าต้อง babysit resume; งบ subagent ต่อ pipeline อาจถึง ~1M tokens
