---
name: reference_ai_board_stale_cards
description: AI board landmines — AI-38 ยังเขียน pgvector/Qdrant ทั้งที่ spike ปิดเป็น Milvus; channel-gateway ไม่มีการ์ดเพราะ E12 ยังไม่เปิด
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e84c3ff-15a2-43f1-989e-32ce9d630f3d
  modified: 2026-08-11T16:53:32.077Z
---

กับดักบนบอร์ด **AI (AI-chatsystem, board 254)** ที่เจอตอนสำรวจ 2026-08-11:

- **AI-38 stale หลัง spike ปิด** — พาดหัวยังเป็น "Vector Infra + Per-Workspace Knowledge Namespace (**pgvector/HNSW**)" และเนื้อการ์ดเขียน "ปรับตามคำแนะนำจาก O1 spike ถ้าแนะนำ **Qdrant**" แต่ **AI-36 (spike O1) = Done** และคำตอบจริงคือ **Milvus** (rag-core ใช้ llama-index + Milvus, 3 collection ตายตัว, ไม่มี concept workspace) ⇒ งานจริงของ AI-38 = "เพิ่ม workspace dimension (partition/filter) ให้ Milvus" ไม่ใช่สร้าง pgvector ใหม่ · **AI-38 อยู่ In Review** = ถ้า dev อ่านพาดหัวจะได้ vector store ผิดตัว (โรคเดียวกับที่ `.claude/rules/design-doc-authority.md` ตั้งมาแก้: spike ปิดแล้วไม่อัปเดตผู้แพ้)
- **E4 = epic AI-5** มี 8 การ์ด: AI-36 (Done) · AI-38 · AI-39 embeddings ingest · AI-41 retrieval ≤150ms · AI-43 KB editor BE · AI-44 Sheet ingest+SSRF egress · AI-46 visibility/ACL+erasure · AI-49 (placeholder Drive/URL)
- **`channel-gateway` (poc) ไม่มีการ์ดโดยเจตนา** — เคาะว่า **E5 ≠ channel-gateway** (ingress BFF vs egress proxy + Python ขัด §5.5) สถานะ 💤 คงเดิม เป็น BFF ของ internal employee assistant และเป็น "ตัวอย่างมีชีวิต" ของ solution template ใน §5.11 ซึ่ง **E12 ยังไม่เปิด epic ใน Jira เลย** ⇒ อยากให้มันมีการ์ดต้องเปิด E12 ก่อน · reusable pattern: SSE relay, rate limit, Google OIDC+Admin SDK groups

ดู [[project_ai_conversation_intelligence]] · [[reference_jira_editissue_adf_breakage]]
