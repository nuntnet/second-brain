---
name: reference_ai_platform_architecture_artifact
description: AI chat platform architecture diagram = artifact 4972126d… ; source file now at docs/ai-platform-architecture.md + mermaid/artifact gotchas
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e84c3ff-15a2-43f1-989e-32ce9d630f3d
  modified: 2026-08-11T16:26:52.704Z
---

**Diagram สถาปัตยกรรม AI Chat Platform** (ที่แผน `docs/ai-chat-assistant-platform-plan.md` §3 อ้างถึง)
= artifact `https://claude.ai/code/artifact/4972126d-7be7-4f0f-8c0a-5812e23568de`

- **เดิมไม่มี source ในรีโปเลย** (แก้ได้จาก session ที่ publish เท่านั้น) — 2026-08-11 สร้าง source ไว้แล้วที่
  **`docs/ai-platform-architecture.md`** → แก้ไฟล์นี้ แล้วเรียก Artifact พร้อม `url=<URL ข้างบน>` = อัปเดตทับ URL เดิม
  (ถ้าไม่ส่ง `url` จาก session อื่น จะได้ artifact ใบใหม่ = ลิงก์ที่ทีมถืออยู่ไม่อัปเดต)
- เป็น **markdown artifact** (mermaid render native ผ่าน ```mermaid fence) ไม่ใช่ HTML page — อย่าแปลงเป็น HTML
- ⚠️ **`&amp;` ใน markdown ถูก escape ซ้อน** → ใน mermaid label โผล่เป็นตัวอักษร `&amp;` (bug นี้ติดมาตั้งแต่ artifact เดิม)
  → ใน label ให้ใช้ `+` หรือคำแทน ห้ามพิมพ์ `&amp;`; `<b>` ใน label ก็เลี่ยง (ใช้ emoji/ข้อความนำแทน)
- ⚠️ **verify ไม่ได้ด้วย in-app browser** (`mcp__Claude_Browser__*`) — artifact เป็น private, browser นั้นไม่ได้ล็อกอิน
  claude.ai จะได้หน้า "Page not found / Sign in" ซึ่ง **ไม่ใช่** หลักฐานว่า artifact พัง → ใช้ `WebFetch` (ใช้ login ของ
  claude.ai ได้) แล้วเช็คว่าเจอ heading/node ใหม่จริง (ระวัง cache 15 นาทีต่อ URL)

เกี่ยวข้อง: [[project_ai_chat_platform_plan]] · [[feedback_verify_as_the_user_sees_it]]
