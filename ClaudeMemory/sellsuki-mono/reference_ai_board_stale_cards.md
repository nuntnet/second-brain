---
name: reference_ai_board_stale_cards
description: AI board — AI-38 vector store เคาะเป็น Milvus แล้ว (แก้การ์ดแล้ว); E12 เปิดเป็น AI-131; channel-gateway ปลดระวาง
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e84c3ff-15a2-43f1-989e-32ce9d630f3d
  modified: 2026-08-12T16:23:39.912Z
---

สถานะจุดที่เคยกำกวมบน **AI board (AI-chatsystem, board 254)** — ปิดแล้วทั้งคู่ (2026-08-12):

- **AI-38 vector store = Milvus** ✅ (user เคาะ 2026-08-12) — การ์ดแก้ครบแล้วทั้ง summary + body: เดิมเขียน "pgvector/HNSW" + "ถ้าแนะนำ Qdrant" ทั้งที่ spike **AI-36 (Done)** ตอบว่า rag-core ใช้ **Milvus ผ่าน llama-index**, 3 collection ตายตัว (internal/public/customer) + collection_router, **ไม่มี concept workspace = single-org** ⇒ งานจริงคือ "เพิ่ม workspace dimension (เสนอ **partition key** ไม่ใช่ collection-per-workspace เพราะ Milvus มีเพดานจำนวน collection) + backfill ข้อมูลเดิม + prod readiness" · **Milvus ไม่มี RLS** ⇒ การบังคับ filter อยู่ชั้นโค้ด ต้องมี guard test กันเรียก client ตรง · `milvus_config` อยู่กับทีม Data (ต้อง sync ก่อนลงมือ)
- **E12 เปิดแล้ว = AI-131** ("Multi-Use-Case Platform + ย้าย Internal Employee Assistant") — **channel-gateway ไม่คงไว้: ปลดระวางหลังย้าย internal assistant มาบน ai-agent/platform** (user เคาะ 2026-08-12, แผน §5.11.1) · epic ยังไม่มีเจ้าภาพและยังไม่มีการ์ดลูก (รอเจ้าภาพ) · gap ที่ไม่มีปลายทางวันนี้: Google OIDC employee login→Kratos/Keto · Web Chat adapter · Discord (ต้องเคาะว่าทำหรือตัด)
- **E4 = epic AI-5** มี 8 การ์ด: AI-36 (Done) · AI-38 · AI-39 ingest · AI-41 retrieval ≤150ms · AI-43 KB editor BE · AI-44 Sheet ingest+SSRF · AI-46 ACL+erasure · AI-49 (placeholder) + **AI-130** (document memory ต่อบุคคล, ใหม่)

ดู [[project_ai_conversation_intelligence]] · [[reference_jira_editissue_adf_breakage]]
