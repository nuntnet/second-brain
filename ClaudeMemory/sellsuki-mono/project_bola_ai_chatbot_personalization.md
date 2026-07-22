---
name: bola-ai-chatbot-personalization
description: "BOLA AI chatbot มีจริงแล้ว (keyword-RAG, no asker context); โจทย์ personalized answer = 4 layer (identity/connector/knowledge/pipeline); gaps สำคัญ"
metadata: 
  node_type: memory
  type: project
  originSessionId: c107f1b5-8d84-48d1-b495-a8743dfcb35b
  modified: 2026-07-22T17:54:01.557Z
---

BOLA AI chatbot (สำรวจ 2026-07-23): implement แล้วเต็ม slice ใน bola-backend — `ai_chatbot_configs` (per-OA), `chat_sessions`, `chat_messages`, `knowledge_base`, `unanswered_questions`; pipeline = `ProcessAIChatMessage` (src/use_case/ai_chatbot.go:515), auto-reply ชนะก่อน chatbot (webhook.go:235-252)

**สถานะจริง:**
- Retrieval = keyword match ไม่ใช่ vector — `embedKnowledgeBaseEntry` เป็น no-op stub (ai_chatbot.go:677); คอลัมน์ `embedding` ไม่เคยถูกเติม
- LLM รู้ line_user_id ของ asker แต่ **ไม่ส่ง identity เข้า prompt** — เห็นแค่ system prompt + KB + history
- LLM API key เก็บ plaintext (TODO ค้าง ai_chatbot.go:48,667); Google provider เลือกได้ใน UI แต่ backend ไม่ implement (default → OpenAI endpoint); FE เรียก `POST /ai-chatbot/test-connection` แต่ backend ไม่มี route
- Identity linking มีแล้ว: `POST /contacts/link` เก็บ `custom_fields["<system>_id"]` (follower.go:783-841) แต่ machine path hardcode oc2plus (route.go:386) + unique index เฉพาะ oc2plus_id (migration 0121); ไม่มี OTP/verify — trust-based
- ไม่มี Google/OAuth client ใดๆ ใน repo; email service = SMTP สำหรับ admin เท่านั้น

**Design ที่เสนอ (โจทย์ "chatbot รู้ว่าผู้ถามคือใคร เช่น วันลาคงเหลือ"):** 4 layers — (1) verified identity linking (LIFF + OTP/Google OAuth), (2) connector framework ต่อ workspace (google_workspace/rest_api/mcp/db), (3) knowledge สองโหมด: pre-ingest RAG (policy) vs live query (per-person facts เช่น leave balance), (4) pipeline inject asker context + tool-use loop โดย identity ต้อง resolve server-side ห้ามให้ LLM เลือกเอง

เกี่ยวข้อง: [[project_bola_contact_profile_model]], [[project_bola_workspace_scoping_bugs]]
