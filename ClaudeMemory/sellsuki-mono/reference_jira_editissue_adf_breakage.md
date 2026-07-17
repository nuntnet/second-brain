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

**createJiraIssue markdown ก็พัง newline (ยืนยัน 2026-07-17, สร้าง BOLA-270):** `createJiraIssue` ด้วย `contentFormat:markdown` เก็บ `\n` เป็น **literal backslash-n** → การ์ดเรนเดอร์เป็นบรรทัดเดียวยาว (heading/table/bullet ไม่ทำงาน). **`editJiraIssue` markdown ไม่พัง newline** (เรนเดอร์ถูก). วิธีแก้: สร้างการ์ดด้วย createJiraIssue (summary + description placeholder สั้นๆ) แล้ว **set description จริงด้วย editJiraIssue อีกที**; หรือสร้างเสร็จ getJiraIssue เช็ค ถ้าเจอ `\n` literal → rewrite ด้วย editJiraIssue.

**Best practice:** (1) การ์ดที่มีรูปฝัง/smartlink — เลี่ยง full rewrite ผ่าน markdown; แก้ target เฉพาะจุด หรือใช้ ADF format. (2) หลัง rewrite ทุกครั้ง **ตรวจ format** โดยเฉพาะใบที่มี media/smartlink. (3) re-edit ผ่าน markdown ซ้ำ = เสี่ยงเกิด escape ซ้ำ. (4) เนื้อหายาว/หลาย section → อย่าใช้ createJiraIssue markdown ตรงๆ ใช้ editJiraIssue set description. ใช้ประกอบ [[project_sukipay_void_rename]].
