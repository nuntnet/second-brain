---
name: reference_jira_editissue_adf_breakage
description: Editing Jira cards via editJiraIssue markdown breaks embedded images/smartlinks + escapes checkboxes — what survives vs breaks
metadata: 
  node_type: memory
  type: reference
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

การแก้ Jira card ผ่าน `editJiraIssue` (Jira MCP) ด้วย markdown → มี **markdown↔ADF round-trip breakage** ที่รู้จากการ rename VOID→COMPENSATING (11 cards):

**รอด (ปลอดภัย):** ตาราง, heading, code block (```), JSON block, GFM checkbox บางกรณี — โครงสร้างหลักไม่พังแม้ตารางเยอะ (PAT-2040 มี 10+ ตาราง ปกติ)

**พัง (ต้องระวัง):**
- **รูปฝัง (embedded media/blob)** → blob URL เพี้ยน (`type=file`→`type=external...url=blob%3A`) รูปแสดงไม่ขึ้น — **แก้ผ่าน markdown ไม่ได้** ต้อง re-attach ใน Jira UI
- **Figma smartlink** → กลายเป็น literal `<custom data-type="smartlink">...</custom>` (คลิกไม่ได้)
- **checkbox `- [ ]`** → บางใบ escape เป็น literal `\[ \]` (ไม่ interactive แต่อ่านได้)
- **escaped quotes/artifacts** — `\\"selector\\"`, `\*`, `\.` โผล่ในบางที่

**Best practice:** (1) การ์ดที่มีรูปฝัง/smartlink — เลี่ยง full rewrite ผ่าน markdown; แก้ target เฉพาะจุด หรือใช้ ADF format. (2) หลัง rewrite ทุกครั้ง **ตรวจ format** โดยเฉพาะใบที่มี media/smartlink. (3) re-edit ผ่าน markdown ซ้ำ = เสี่ยงเกิด escape ซ้ำ. ใช้ประกอบ [[project_sukipay_void_rename]].
