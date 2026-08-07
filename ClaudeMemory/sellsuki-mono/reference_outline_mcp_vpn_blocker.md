---
name: reference-outline-mcp-vpn-blocker
description: Outline MCP (docs.sellsuki.com) ต่อไม่ได้ถ้าไม่อยู่บน VPN — โฮสต์ภายใน 10.x; ใช้ publish-to-outline.py แทน
metadata: 
  node_type: memory
  type: reference
  originSessionId: 60159299-7c45-49ce-8985-bacd71095aa0
  modified: 2026-08-07T16:51:13.494Z
---

`outline` MCP server ชี้ไปที่ `https://mcp-outline.internal.production.sellsuki.com/mcp`
ซึ่ง resolve เป็น **internal ELB 10.111.9.75** → connection timeout ถ้าไม่ได้อยู่บน VPN บริษัท
(Tailscale ไม่ช่วย — คนละ network; Tailscale ใช้กับ NAS ds1 เท่านั้น ดู [[reference-nas-ds1]])

`docs.sellsuki.com` เอง (Outline UI) เข้าได้ปกติ (HTTP 200) — บล็อกเฉพาะ MCP endpoint ภายใน

**ข้อควรรู้:** ถ้า session เริ่มตอน VPN ยังไม่ขึ้น MCP tools จะ**ไม่โผล่แม้ต่อ VPN ทีหลัง**
(`claude mcp list` ขึ้น ✔ Connected แต่ ToolSearch หาไม่เจอ — registry ถูก build ตอนเริ่ม session)
→ ทางออกที่ใช้ได้จริง: **ยิง MCP ตรงผ่าน HTTP** — server นี้ **ไม่ต้อง auth** (อยู่ใน internal network)
POST streamable-HTTP: `initialize` → `notifications/initialized` → `tools/call`
เก็บ `Mcp-Session-Id` จาก response header · response เป็น SSE (`data: {...}`)
· **ต้องใช้ certifi** ไม่งั้น SSLCertVerificationError

ตัวอย่างที่ทำงานแล้ว: `docs/product-kb/publish-to-outline.py`
(dry-run เป็นค่าเริ่มต้น, idempotent ผ่าน `outline-map.json`, ไม่ต้องใช้ API token)

## กับดัก 3 ข้อของ Outline MCP (เจอจริง 2026-08-07 — เช็คทุกครั้งที่ publish)

1. **collection ที่สร้างผ่าน MCP = private ของ bot account** (`permission: null`)
   คนในบริษัท**มองไม่เห็นเลย** (`authorization_error` ตอนเปิด doc) และ MCP `update_collection`
   ตั้ง permission ไม่ได้ (รับแค่ name/description/color)
   → แก้ด้วย session ของ user เอง: `POST /api/collections.update {id, permission:'read'}`
   (`read` = อ่านอย่างเดียว เหมาะกับ doc ที่ต้นทางอยู่ใน repo กัน drift)
2. **MCP ไม่คืน URL ของเอกสาร คืนแค่ UUID — และ `/doc/<uuid>` ใช้ไม่ได้ (Not Found)**
   URL จริงเป็น slug + urlId เช่น `/doc/0-portfolio-overview-ecosystem-x7YHAwCtzZ`
   → เอา URL จริงจาก `POST /api/documents.info {id}` → `data.url` (ต้องใช้ browser ที่ล็อกอินอยู่)
3. **`move_document` ไม่มีพารามิเตอร์ index → เรียง sidebar ไม่ได้ผ่าน MCP**
   (หน้าเรียงตาม insert ล่าสุดขึ้นบน) ต้องลากเองใน UI หรือตั้ง collection sort เป็น alphabetical

ข้อ 1–2 ทำให้ "publish สำเร็จ" ตาม log **แต่ user เปิดไม่ได้จริง** — ต้อง verify ใน browser
ที่ล็อกอินอยู่เสมอ (Claude in Chrome) อย่าเชื่อ HTTP 200 จาก curl เพราะ SPA คืน 200 ทุก route

รายชื่อ collection id ของ workspace อยู่ใน `.claude/rules/outline-collections.md`
(ของ repo boilerplate) — ตรวจ 2026-08-07 แล้ว: id ยังตรง แต่มี collection ใหม่เพิ่มที่ไฟล์นั้นไม่มี

เกี่ยวข้อง: [[project-sellsuki-product-kb]] · [[project-control-tower]]
