---
name: reference_ai_chat_admin_mock_preview_5192_no_login
description: เปิด AI Chat console แบบ mock ล้วนผ่าน preview_start name=web-ai-chat-admin-mock (port 5192) ไม่ต้อง login/backend · workspace ws_demo_default
metadata:
  type: reference
---

`mcp__Claude_Browser__preview_start {name: "web-ai-chat-admin-mock"}` (ใน /Users/nunt/sellsuki_mono/.claude/launch.json) รัน `VITE_USE_MOCKS=true npx vite --port 5192` — MSW stub Kratos whoami เอง ไม่ต้อง overmind, Kratos, chat-core หรือ cookie · เข้า `http://localhost:5192/w/ws_demo_default[/setup|/dashboard|/kb|/settings/persona]` · ปุ่ม TH/EN อยู่ใน rail ซ้ายล่าง · ใช้ resize_window 1440×900 / 390×844 · ปิดด้วย preview_stop

ใช้ได้กับ layout, contrast, geometry, a11y tree · ห้ามใช้เป็นหลักฐานเรื่องข้อมูล/ข้อความ ดู [[feedback_mock_mode_ui_review_check_real_adapter_before_calling_defect]] · ถ้าต้องการ stack จริงบน 5191 ดู [[reference_local_ai_chat_console_open_on_localhost_with_seed_identity]]
