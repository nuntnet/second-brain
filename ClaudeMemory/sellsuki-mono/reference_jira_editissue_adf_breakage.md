---
name: reference_jira_editissue_adf_breakage
description: Editing Jira cards via editJiraIssue markdown breaks embedded images/smartlinks + escapes checkboxes — what survives vs breaks
metadata: 
  node_type: memory
  type: reference
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
  modified: 2026-08-12T18:47:59.247Z
---

การแก้ Jira card ผ่าน `editJiraIssue` (Jira MCP) ด้วย markdown → มี **markdown↔ADF round-trip breakage** ที่รู้จากการ rename VOID→COMPENSATING (11 cards):

**รอด (ปลอดภัย):** ตาราง, heading, code block (```), JSON block, GFM checkbox บางกรณี — โครงสร้างหลักไม่พังแม้ตารางเยอะ (PAT-2040 มี 10+ ตาราง ปกติ)

**พัง (ต้องระวัง):**
- **รูปฝัง (embedded media/blob)** → blob URL เพี้ยน (`type=file`→`type=external...url=blob%3A`) รูปแสดงไม่ขึ้น — **แก้ผ่าน markdown ไม่ได้** ต้อง re-attach ใน Jira UI
- **Figma smartlink** → กลายเป็น literal `<custom data-type="smartlink">...</custom>` (คลิกไม่ได้)
- **checkbox `- [ ]`** → บางใบ escape เป็น literal `\[ \]` (ไม่ interactive แต่อ่านได้)
- **escaped quotes/artifacts** — `\\"selector\\"`, `\*`, `\.` โผล่ในบางที่

**createJiraIssue markdown ก็พัง newline (ยืนยัน 2026-07-17, สร้าง BOLA-270):** `createJiraIssue` ด้วย `contentFormat:markdown` เก็บ `\n` เป็น **literal backslash-n** → การ์ดเรนเดอร์เป็นบรรทัดเดียวยาว (heading/table/bullet ไม่ทำงาน). **`editJiraIssue` markdown ไม่พัง newline** (เรนเดอร์ถูก). วิธีแก้: สร้างการ์ดด้วย createJiraIssue (summary + description placeholder สั้นๆ) แล้ว **set description จริงด้วย editJiraIssue อีกที**; หรือสร้างเสร็จ getJiraIssue เช็ค ถ้าเจอ `\n` literal → rewrite ด้วย editJiraIssue.

**🔴 SILENT CONTENT LOSS — blockquote ซ้อนใน list item (ยืนยัน 2026-08-07 บน OC-4415):** ถ้าเขียน `> ...` (blockquote) **ซ้อนอยู่ใต้ bullet/AC item** → markdown→ADF **ตัด bullet ทั้งข้อทิ้งเงียบ ๆ** ไม่มี error, tool คืน success ปกติ (AC-03 หายทั้งบรรทัด). ต่างจากเคสอื่นตรงที่ **ไม่เห็นร่องรอย** ต้อง `getJiraIssue` กลับมาอ่านถึงจะรู้. **แก้:** เขียน note เป็น plain text ในบรรทัดเดียวกับ bullet (ห้าม `>` ซ้อน) — blockquote ระดับบนสุด (ไม่ซ้อนใน list) ยังใช้ได้ปกติ.
→ **หลัง editJiraIssue ทุกครั้งที่มี list ยาว ควร re-fetch นับจำนวน AC เทียบ** ไม่ใช่เชื่อ success response

**`searchJiraIssuesUsingJql` payload บวมมาก (2026-08-11):** แม้ส่ง `fields: ["summary","status"]` ผลลัพธ์ ~10-20 ใบ
ก็ทะลุ token cap (169k chars) เพราะแนบ project/issuetype/avatar blob + description เต็มมาทุกใบ → tool จะ save เป็นไฟล์
แล้วบังคับให้อ่านทั้งไฟล์. **วิธีที่เร็ว:** ปล่อยให้มัน save แล้ว `jq -r '.issues.nodes[] | "\(.key)\t\(.fields.status.name)\t\(.fields.summary)"' <file>`
(ใช้ `/usr/bin/jq` เลี่ยง rtk) — ได้ตารางย่อในไม่กี่ร้อย token แทนที่จะอ่านไฟล์ทั้งก้อน

**🔴 `description ~ "x"` ให้ false positive จนใช้เช็ค "การ์ดถูกแก้แล้วหรือยัง" ไม่ได้ (ยืนยัน 2026-08-12/13):** JQL text search tokenize + ตัด underscore ⇒ `description ~ "case_id"` ไป match คำไทย **"เคส"** และ column header **"Test Case"** → สรุปผิดว่าการ์ด 3 ใบ (AI-16/47/95) ถูกอัปเดตแล้วทั้งที่ยังไม่เคยแตะ **แล้วเอาไปแบ่งงาน sub-agent ผิด (ตกไป 2 ใบ)** · `~ "§5.13"` ก็ match ไม่ได้ (อักขระพิเศษ+เลขทศนิยม) · **วิธีที่เชื่อได้:** probe ด้วย token ที่ไม่มีทางเกิดเองในการ์ดเดิม (`denormalized`, `attribution`, `subject_ref`, `unmapped`) หรือ getJiraIssue มา grep เอง

**🟡 `{"errors":{"issuelinks":"'SET' operation is not supported."}}` ตอน edit ที่ส่งแค่ `description` = TRANSIENT (ยืนยัน 2026-08-13):** เจอครั้งแรกบน AI-52 แล้ว "ผ่านหลังเอา issue key ออกจากตาราง Dependencies" → **สรุปสาเหตุผิด** · รอบยืนยันภายหลังใส่ key กลับทั้ง 4 คีย์ (plain text ไม่ใช่ link syntax) **ผ่านรอบเดียว ไม่มี error** ⇒ **ไม่ใช่เพราะ issue key ในเนื้อ** เป็นอาการชั่วคราวของ MCP · **วิธีรับมือ: retry ก่อน อย่าตัดเนื้อหาการ์ดทิ้งเพื่อเลี่ยง error** (การตัด key ออกทำให้ตาราง Dependencies กดตามไม่ได้ = เสียของฟรี)

**🔴 `{"errors":{"description":"CONTENT_LIMIT_EXCEEDED"}}` — การ์ดที่โตเกิน ~30k ตัวอักษรแก้ผ่าน MCP ไม่ได้อีก (ยืนยัน 2026-08-13 บน AI-126):** เพดาน description ของ Jira = **32,767 ตัวอักษร** · AI-126 อยู่ที่ ~30k ⇒ การ **เพิ่ม** เนื้อ ~2.5k fail ทั้งก้อน (ไม่ใช่ตัดท้าย — reject ทั้ง request) · **แต่การ swap ที่ทำให้สั้นลงยังทำได้** ⇒ แยกให้ออกระหว่าง "แก้ไม่ได้" กับ "เพิ่มไม่ได้" ก่อนสรุปว่าติดตาย · **นัยเชิง DoR**: การ์ด 30k ตัวอักษรคือกลิ่นว่าเนื้อซ้ำ/ควร split — พอชนเพดานแล้วจะแก้อะไรต่อไม่ได้เลยจนกว่าจะมีคนตัดสินใจตัด (เจ้าของการ์ดต้องตัด ไม่ใช่ agent ตัดเอง)

**🔴 ห้ามส่งข้อความไทยเป็น `\uXXXX` escape ใน payload ของ editJiraIssue (ยืนยัน 2026-08-13 — เจอ 4 agent ใน session เดียว):** escape ผิดตัวเดียวได้คำเพี้ยนที่อ่านผ่านตาไม่เห็น — พบจริง: `กฎ`→`กฏ`, `เกณฑ์`→`เกณฏ์`, `บริโภค`→`บริโกค`, `การคำนวณ`→`การคำนวຓ` (อักษรลาว!), `เรื่องหนึ่ง`→`เรื่อหนึ่ง`, `หนึ่ง`→`หนี่ง` · สระ/วรรณยุกต์หายแบบเงียบและ Jira ไม่ error ⇒ **เขียนไทยตรง ๆ ใน payload** และถ้าจำเป็นต้อง escape ต้อง re-fetch มาอ่านคำที่แก้ทีละคำ

**🔴 response ตอบกลับเป็นการ์ดคนละใบ (2026-08-12):** `editJiraIssue` ของ AI-45 คืน payload ของ AI-133 · `createIssueLink` เคยคืน getJiraIssue ของ BOLA-313 ที่ไม่เกี่ยวเลย — **ของถูกเขียนถูกใบ** เป็น response mix-up ⇒ ยิ่งตอกย้ำว่าต้อง re-fetch ใบที่ตั้งใจแก้แยกเสมอ

**Best practice:** (0) **re-fetch verify เสมอ** ไม่ใช่แค่ตอนมีรูป — content loss แบบเงียบมีจริง (ดูข้อบน). (1) การ์ดที่มีรูปฝัง/smartlink — เลี่ยง full rewrite ผ่าน markdown; แก้ target เฉพาะจุด หรือใช้ ADF format. (2) หลัง rewrite ทุกครั้ง **ตรวจ format** โดยเฉพาะใบที่มี media/smartlink. (3) re-edit ผ่าน markdown ซ้ำ = เสี่ยงเกิด escape ซ้ำ. (4) เนื้อหายาว/หลาย section → อย่าใช้ createJiraIssue markdown ตรงๆ ใช้ editJiraIssue set description. ใช้ประกอบ [[project_sukipay_void_rename]].
