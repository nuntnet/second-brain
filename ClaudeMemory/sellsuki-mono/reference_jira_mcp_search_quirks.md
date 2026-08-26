---
name: jira-mcp-search-quirks
description: "Jira MCP searchJiraIssuesUsingJql — parallel-call collision bug, fields param ignored, tiny pages; summary field HTML-escapes"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 22bc8e4d-cfe5-415c-8c23-d4d838fba867
  modified: 2026-08-14T12:12:23.521Z
---

Quirks ของ Jira MCP tools (server d4aa0ce4…, พบ 2026-08-14 ตอนไล่ทั้ง board AI 141 ใบ):

1. **Parallel-call collision**: ยิง `searchJiraIssuesUsingJql` หลาย call ใน message เดียว → สอง query ต่างกันได้ข้อมูลชุดเดียวกัน (ผิด) — ต้องเรียก **sequential เท่านั้น**
2. **`fields` param ถูกเพิกเฉย**: ขอแค่ summary/status ก็ยังได้ description เต็มกลับมา → page ละ ~5 issues, ผลลัพธ์ 75k chars ล้น context เร็วมาก
3. วิธีที่เวิร์ค: **delegate การ paginate ให้ subagent** เขียน TSV ลง scratchpad, หรือใช้ `searchResultMode: "count"` + `key = X AND text ~ "term"` เช็คว่าการ์ดใบหนึ่งมีคำนั้นหรือยัง (ถูกมาก — ใช้ verify ว่า card edits ถูก apply แล้ว)
4. `createJiraIssue` **HTML-escape อักขระ `<` ใน summary** (`<30 นาที` → `&amp;lt;30`) — เลี่ยงอักขระพิเศษใน summary; แก้ทีหลังด้วย editJiraIssue เฉพาะ field summary ได้ (ไม่แตะ description จะไม่โดนบั๊ก ADF ตาม [[jira-editissue-adf-breakage]])

**How to apply:** งานไหนต้องกวาด Jira board ทั้งบอร์ด → spawn subagent + sequential calls + TSV; งาน verify เนื้อหาการ์ด → count-mode text ~ query

5. **Issue links: ทิศทางกลับด้าน + ลบไม่ได้** (2026-08-26, cluster OC-4362/4407) — `createIssueLink` ผ่าน MCP บันทึกทิศกลับด้านได้ (การ์ดที่ควรเป็น *is blocked by* กลายเป็น *blocks*) และ **ไม่มี tool สำหรับลบ link** → เจอแล้ว 2 ครั้งค้างเป็นเดือน (OC-4407 link 23380 ตั้งแต่ 2026-08-07, OC-4362 link 23390). ต้อง **verify ทิศทางด้วย getJiraIssue หลังสร้างทุกครั้ง** และถ้าผิดต้องให้คนลบใน Jira UI เอง — อย่าปล่อยไว้ เพราะ dependency graph ที่กลับด้านทำให้ prioritizer จัดลำดับผิด
6. 2026-08-26: `fields` param **ทำงานแล้ว** ทั้ง getJiraIssue และ searchJiraIssuesUsingJql (ต่างจากข้อ 2 ที่พบ 2026-08-14) — แต่ nested `parent` ยังคืน object เต็มเสมอ ทำให้ผลลัพธ์บวมอยู่ดีเมื่อ query หลายสิบใบ
