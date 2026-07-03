---
name: reference-pat-board-sprints
description: PAT sprint ground truth = Jira board 71 (JQL openSprints/futureSprints + customfield_10020) — ห้ามใช้ epic-index เป็น sprint state
metadata: 
  node_type: memory
  type: reference
  originSessionId: b0b4a24f-96f9-41ef-b15d-18e86ad938e1
---

**Sprint state ของ PAT ต้อง query จาก Jira เท่านั้น** — `project = PAT AND sprint in openSprints()` / `futureSprints()`, sprint field = `customfield_10020` (board 71, ชื่อ "PAT Sprint NNN"). ระวัง: ผลรวม description เสมอ → overflow; jq บนไฟล์ auto-saved

**Why:** เคยพลาดหนัก (2026-07-03) — สร้าง sprint plan จาก epic-index.md ที่ maintain มือ ปรากฏว่า sprint label ผิดยกชุด และตัวตนการ์ดผิด (PAT-2038 จริง = "ยกเลิก Order" ไม่ใช่ VOID; PAT-2036 จริง = "Tab รอคืนเงิน") user ต้องท้วง

**How to apply:** ก่อน sprint planning ทุกครั้ง query board ก่อน. Snapshot 2026-07-03: S98 active (SukiPay refund/compensation + PAT-2295 payment integration), S101 เริ่ม 2026-08-10 บวม 27 ใบ (foundation+slices+LINK_BILL ทั้งชุด), S103 = dumping 39 ใบ (card gateway ladder 2070–2095), S99/S102 เบา (7/6). Batch-3 cards (2503–2531) + PAT-2479 ยังไม่มี sprint. Cadence ~2 สัปดาห์/sprint. เกี่ยวข้อง [[project-oms2-plan-gaps-2026q3]] [[project-pat-epic-links-unwired]]
