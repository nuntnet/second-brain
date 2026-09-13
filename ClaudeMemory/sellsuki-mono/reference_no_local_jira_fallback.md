---
name: reference_no_local_jira_fallback
description: "เมื่อ Atlassian MCP connector หลุดกลางเซสชัน ไม่มีทางสำรองบนเครื่องนี้เลย — ไม่มี jira CLI, ไม่มี credential, internal mcp-atlassian ต้องต่อ VPN; อย่าเสียคอลไปหา ให้บอกผู้ใช้เปิด connector"
metadata: 
  node_type: memory
  type: reference
  originSessionId: d0b39379-feaa-4fd5-b115-18617c202159
  modified: 2026-09-13T14:40:45.381Z
---

**Atlassian MCP connector หลุดกลางเซสชันได้ และไม่มี fallback บนเครื่องนี้** (ยืนยัน 2026-09-13 ตอนเขียน BOLA-328/OC-4523)

อาการ: connector หายไปทั้งตัวพร้อม system-reminder ว่า "MCP server removed from the configuration" — ต่างจากสถานะ `failed` ตรงที่ **`reconnect_session_connector` ใช้ไม่ได้** (มันรับเฉพาะ kind `connector` ที่ยัง failed อยู่) และ `session_connectors_status` จะไม่แสดงแถว Atlassian เลย ผู้ใช้ต้องเปิดเองจาก Settings → Connectors แล้วบอกเรา

ทางสำรองที่ **ไม่มี** (เช็คแล้วทั้งหมด อย่าเสียคอลไปหาซ้ำ):
- ไม่มี `jira` / `acli` / `atlassian` CLI ใน PATH
- ไม่มี env var ขึ้นต้น `JIRA_` หรือ `ATLASSIAN_`
- `~/.netrc` ไม่มี machine ของ atlassian
- `backend/space-go/.mcp.json` ชี้ `https://mcp-atlassian.internal.production.sellsuki.com/sse` — **timeout** จากเครือข่ายบ้าน ต้องต่อ VPN ก่อน (เข้าคู่กับ [[reference_outline_mcp_vpn_blocker]])

**วิธีรับมือที่เสียเวลาน้อยสุด:** compose เนื้อการ์ดลงไฟล์ใน scratchpad ให้เสร็จ → `SendUserFile` ส่งให้ผู้ใช้ → บอกว่าต้องเปิด connector แล้วจะยิงต่อ ผู้ใช้เปิดให้ในเทิร์นถัดไปได้เร็ว และงานที่ร่างไว้ไม่หาย (ทำจริงในเซสชันนี้ ร่าง 2 ใบรอดครบ)

⚠️ ถ้าการ์ดถูก **สร้างไปแล้ว** ก่อน connector หลุด (createJiraIssue ผ่าน แต่ editJiraIssue ยังไม่ได้ยิง) จะเหลือการ์ด description เป็น placeholder ค้างบนบอร์ด — จำเลขการ์ดไว้แล้วยิง description ต่อทันทีที่ connector กลับมา อย่าสร้างใบใหม่ ดู [[reference_jira_editissue_adf_breakage]] เรื่อง pattern create-placeholder-แล้ว-edit
