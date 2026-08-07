---
name: reference-outline-mcp-vpn-blocker
description: Outline MCP (docs.sellsuki.com) ต่อไม่ได้ถ้าไม่อยู่บน VPN — โฮสต์ภายใน 10.x; ใช้ publish-to-outline.py แทน
metadata: 
  node_type: memory
  type: reference
  originSessionId: 60159299-7c45-49ce-8985-bacd71095aa0
  modified: 2026-08-07T16:14:15.748Z
---

`outline` MCP server ชี้ไปที่ `https://mcp-outline.internal.production.sellsuki.com/mcp`
ซึ่ง resolve เป็น **internal ELB 10.111.9.75** → connection timeout ถ้าไม่ได้อยู่บน VPN บริษัท
(Tailscale ไม่ช่วย — คนละ network; Tailscale ใช้กับ NAS ds1 เท่านั้น ดู [[reference-nas-ds1]])

`docs.sellsuki.com` เอง (Outline UI) เข้าได้ปกติ (HTTP 200) — บล็อกเฉพาะ MCP endpoint ภายใน

**ทางออกเมื่อต้อง publish เอกสารขึ้น Outline โดยไม่มี MCP:**
`docs/product-kb/publish-to-outline.py` — ใช้ Outline REST API ตรง
(`OUTLINE_API_TOKEN` + `--collection <id>`), dry-run เป็นค่าเริ่มต้น, idempotent
ผ่าน `outline-map.json`, rewrite ลิงก์ระหว่างไฟล์เป็น Outline URL ให้

รายชื่อ collection id ของ workspace อยู่ใน `.claude/rules/outline-collections.md`
(ของ repo boilerplate) — ยังไม่เคยตรวจว่า id ปัจจุบันตรงไหม

เกี่ยวข้อง: [[project-sellsuki-product-kb]] · [[project-control-tower]]
