---
name: reference_jira_board_sweep_via_saved_results
description: "กวาด Jira ทั้ง sprint/บอร์ดผ่าน MCP: ผลใหญ่ถูกเซฟเป็นไฟล์ให้ parse เอง (python strict=False ไม่ใช่ jq) · ชื่อ sprint OC คือ \"OC Sprint N\" · connector ขึ้น connected แต่ตอบ 'server isn't responding'/'security policy' ชั่วคราว รอ 1-2 นาทีแล้วยิงใหม่"
metadata:
  type: reference
---

พบ 2026-09-22 ตอนรีวิว OC Sprint 130 (73 ใบ) — วิธีที่เวิร์คโดยไม่ต้อง spawn subagent ไล่หน้า:

1. **ผลลัพธ์ที่ใหญ่เกิน context ไม่หาย** — harness เซฟเป็นไฟล์ที่
   `~/.claude/projects/<proj>/<session>/tool-results/mcp-…searchJiraIssuesUsingJql-<ts>.txt`
   แล้วบอก path กลับมา ⇒ ขอ `maxResults: 50` + fields เต็ม (description, comment, issuelinks)
   ได้เลย ไฟล์หนึ่ง ~850k chars ก็รับได้ แล้วค่อยแยกเป็นไฟล์ต่อการ์ดเอง
2. **ห้าม parse ด้วย jq** — description แบบ markdown มี control char ดิบ (tab) ใน string
   → jq พังทุกไฟล์ ใช้ `python3 json.loads(raw, strict=False)` ผ่านทุกไฟล์
3. `fields` param **ทำงาน** (ยืนยันอีกครั้ง) — ขอแค่ summary/status/customfield_10020 ได้ผล
   50 ใบ ≈ 75k chars; ถ้าใส่ description จะ ≈ 850k
4. **ชื่อ sprint ของบอร์ด OC = `"OC Sprint 130"`** (`sprint = "OC Sprint 130"` ใช้ได้;
   `"OC2Plus Sprint 130"` คืน 0 เงียบ ๆ) · `sprint in futureSprints()` /
   `openSprints()` ใช้ได้ · ณ 2026-09-22 บอร์ด OC **ไม่มี sprint active เลย** — 129 และ 130
   เป็น future ทั้งคู่ทั้งที่ 129 มีการ์ด Ready to test 31 ใบ
5. **error ชั่วคราวของ Atlassian connector**: `session_connectors_status` บอก connected แต่ทุกคอล
   ตอบ "The connector's server isn't responding" ~5 นาที แล้วเปลี่ยนเป็น "security policy
   restricts access" 1 ครั้ง แล้วก็กลับมาปกติเอง — ไม่ใช่ connector หลุด (ต่างจาก
   [[reference_no_local_jira_fallback]]) รอแล้วยิงซ้ำ อย่าไปหา fallback
6. glab อยู่ในเครื่องและ login แล้ว (`glab auth status`) — `glab mr list --all -F json` ต่อรีโป
   แล้ว regex `OC-\d{4}` จาก title/branch/description ได้ตาราง MR↔การ์ดครบทุกรีโปใน 1 นาที
   ⚠️ `backend/file-service-cronjob` git dir เพี้ยน โชว์ commit ของทุกรีโป — ตัดออกจากการ sweep

Related: [[jira-mcp-search-quirks]], [[reference_jira_mcp_crosses_responses_between_sessions]]
