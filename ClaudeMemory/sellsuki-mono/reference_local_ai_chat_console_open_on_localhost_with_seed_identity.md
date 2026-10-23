---
name: reference_local_ai_chat_console_open_on_localhost_with_seed_identity
description: "วิธีเปิด ai-chat-admin console บน local แบบ login ได้โดยไม่พิมพ์รหัสบนโดเมน .local: รัน overmind ด้วย -l …,web-ai-chat-admin,kratos-ui → http://localhost:5191 (vite proxy /self-service,/sessions→:4433, /v1→:8099 จึงใช้ cookie บน localhost) → login ด้วย curl Kratos browser flow ที่ 127.0.0.1:4433 ด้วย identity ใน .claude/verify/.local/credentials.json (seed-verify.sh) → cookie sellsuki_session มี Domain=sellsuki.local จึงไม่เข้า jar ของ curl ต้องอ่านจาก Set-Cookie → วางเป็น cookie ของ localhost ในแท็บผ่านไฟล์ชั่วคราวใน apps/admin/public/ (ลบทิ้ง) · route ของหน้า = /w/<ws>/settings/<page> (ไม่ใช่ /w/<ws>/<page>)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-23T12:10:34.201Z
---

ใช้จริง 2026-10-23 (AI-303 AC7 — "หน้าที่ไม่เคยเปิดยังไม่ถือว่าทำแล้ว"):

1. stack: `overmind quit; rm -f .overmind.sock; overmind start -f Procfile.ai-mvp -l ai-agent,chat-core,role-perm,web-ai-chat-admin,kratos-ui -N -D` (console = `web-ai-chat-admin` :5191; `kratos-ui` :4455 ไม่จำเป็นถ้า login ด้วย curl) · ถ้า branch ของ admin-fe ยังไม่อยู่ใน main checkout: `scripts/local-stack.sh add frontend/ai-chat-admin-frontend <branch>`
2. เปิด **http://localhost:5191** ไม่ใช่ chat.sellsuki.local — vite proxy (`apps/admin/vite.config.ts`) ส่ง `/self-service`, `/sessions` → :4433 และ `/v1` → :8099 ดังนั้น cookie ที่ตั้งบน host `localhost` ใช้ได้ทั้ง Kratos whoami และ chat-core · แอปจะ redirect ไป https://accounts.sellsuki.local/login ถ้าไม่มี session (โดเมน .local ไม่อยู่ในรายการที่ให้พิมพ์รหัสได้ → อย่าพิมพ์ที่นั่น)
3. login ด้วย curl บน localhost: `GET 127.0.0.1:4433/self-service/login/browser` (อ่าน flow id จาก Location) → `GET /self-service/login/flows?id=` (csrf_token) → `POST /self-service/login?flow=` JSON `{method:password, identifier, password, csrf_token}` + `Accept: application/json` → 200 · credentials อ่านจาก `.claude/verify/.local/credentials.json` key `kratos` (email/password/identity_id ของ `scripts/seed-verify.sh`; gitignored) ใน python โดยไม่ print
4. cookie `sellsuki_session` ถูกตั้ง `Domain=sellsuki.local` → curl -c ไม่เก็บ (host ไม่ตรง) ต้อง `-D hdr` แล้ว parse `Set-Cookie` เอง
5. ใส่ให้แท็บ: เขียน `{"t":"<token>"}` ลง `frontend/ai-chat-admin-frontend/apps/admin/public/__x.json` (vite เสิร์ฟ static) → navigate แท็บไป `http://localhost:5191/__x.json` (path ที่ SPA ไม่ redirect) → javascript_tool `fetch('/__x.json')… document.cookie='sellsuki_session='+t+'; path=/'` → ลบไฟล์ → navigate ไปหน้า
6. route: `/w/<workspaceId>/settings/anti-abuse` (settings ทุกหน้าอยู่ใต้ `/settings/`; `/w/<ws>/anti-abuse` = 404 ของ react-router)
7. accessibility tree ของ pane ไม่เห็น input ในหน้า (DS web component) → ใช้ screenshot + coordinate (triple_click เลือกค่า, type, Tab ให้ validate, ปุ่ม Save เลื่อนตำแหน่งเมื่อข้อความ error หาย — ถ่ายใหม่ก่อนคลิก) · ตรวจผลจาก `read_network_requests urlPattern=anti-abuse-config` (PATCH 200 + GET refetch) และ DB

เกี่ยวข้อง: [[reference_staging_ai_chat_console_drive_recipe]] (staging ให้ PO ล็อกอินใน pane เอง), [[project_test_users_as_infrastructure]], [[feedback_verify_as_the_user_sees_it]]
