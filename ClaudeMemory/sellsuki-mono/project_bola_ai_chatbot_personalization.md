---
name: bola-ai-chatbot-personalization
description: "BOLA AI chatbot มีจริงแล้ว (keyword-RAG, no asker context); โจทย์ personalized answer = 4 layer (identity/connector/knowledge/pipeline); gaps สำคัญ"
metadata: 
  node_type: memory
  type: project
  originSessionId: c107f1b5-8d84-48d1-b495-a8743dfcb35b
  modified: 2026-07-23T01:01:28.215Z
---

BOLA AI chatbot (สำรวจ 2026-07-23): implement แล้วเต็ม slice ใน bola-backend — `ai_chatbot_configs` (per-OA), `chat_sessions`, `chat_messages`, `knowledge_base`, `unanswered_questions`; pipeline = `ProcessAIChatMessage` (src/use_case/ai_chatbot.go:515), auto-reply ชนะก่อน chatbot (webhook.go:235-252)

**สถานะจริง:**
- Retrieval = keyword match ไม่ใช่ vector — `embedKnowledgeBaseEntry` เป็น no-op stub (ai_chatbot.go:677); คอลัมน์ `embedding` ไม่เคยถูกเติม
- LLM รู้ line_user_id ของ asker แต่ **ไม่ส่ง identity เข้า prompt** — เห็นแค่ system prompt + KB + history
- LLM API key เก็บ plaintext (TODO ค้าง ai_chatbot.go:48,667); Google provider เลือกได้ใน UI แต่ backend ไม่ implement (default → OpenAI endpoint); FE เรียก `POST /ai-chatbot/test-connection` แต่ backend ไม่มี route
- Identity linking มีแล้ว: `POST /contacts/link` เก็บ `custom_fields["<system>_id"]` (follower.go:783-841) แต่ machine path hardcode oc2plus (route.go:386) + unique index เฉพาะ oc2plus_id (migration 0121); ไม่มี OTP/verify — trust-based
- ไม่มี Google/OAuth client ใดๆ ใน repo; email service = SMTP สำหรับ admin เท่านั้น

**Design ที่เสนอ (โจทย์ "chatbot รู้ว่าผู้ถามคือใคร เช่น วันลาคงเหลือ"):** 4 layers — (1) verified identity linking (LIFF + OTP/Google OAuth), (2) connector framework ต่อ workspace (google_workspace/rest_api/mcp/db), (3) knowledge สองโหมด: pre-ingest RAG (policy) vs live query (per-person facts เช่น leave balance), (4) pipeline inject asker context + tool-use loop โดย identity ต้อง resolve server-side ห้ามให้ LLM เลือกเอง

**Cross-check กับ BOLA-294 (reply token epic):** ไม่ทับ scope (294 = "ส่งยังไง", personalization = "ตอบอะไร") แต่ 4 ข้อต้อง align — (1) BOLA-297 เปลี่ยน signature `ProcessAIChatMessage` (รับ replyToken) + จุดส่งทั้ง 3 → ให้ 297 land ก่อน Phase 1; (2) tool-use latency จะหลุด reply window → policy: RAG-only ตอบด้วย reply, tool path ใช้ token ส่ง ack แล้ว push คำตอบจริง; (3) send ใหม่ทุกตัว (tool answer, LIFF link invite) ต้องเรียก shared send-outcome emitter ของ 297 + metric 298 จะเห็น push_fallback สูงขึ้นโดย design; (4) requirement ใหม่: personal answer ห้าม reply เข้า group — chat_type group/room ต้องปิด personal tools หรือเบี่ยงไป push 1:1

**Connector ลำดับที่เคาะ:** oc2plus → generic rest_api → google_workspace — oc2plus เป็นตัวแรกเพราะ identity link มีแล้ว (LIFF register OC-4245 → /contacts/link, verified), ฝั่ง OC2Plus มี 3rd-party API surface กำลังสร้าง (OC-2275 WS1-A ต้อง land ก่อน = dependency), use case "แต้ม/tier/คูปอง" ขายได้ทันที; ขาใหม่ BOLA→OC2Plus ยังไม่มี client — การ์ด serve ฝั่ง OC board, การ์ดเรียกฝั่ง BOLA board

เกี่ยวข้อง: [[project_bola_contact_profile_model]], [[project_bola_workspace_scoping_bugs]], [[project_bola_reply_token_epic]], [[project_oc2plus_3rdparty_apikey_gap]], [[project_oc_bola_domain_boundary]]
