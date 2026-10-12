---
name: project_bola_follower_idp_epic
description: "BOLA-337 epic (2026-10-11) — ผู้ติดตาม LINE OA ยืนยันตัวตนกับ IdP ขององค์กร (OIDC ต่อ OA) แล้ว rich menu / LIFF / chatbot ตามสิทธิ์ · การ์ด BOLA-338/339/340/341 · เคาะแล้ว: สิทธิ์มาได้ทั้งจาก claims และแอดมิน · ใช้ library OIDC ร่วมกับแอดมินแต่ config แยกต่อ OA"
metadata:
  node_type: memory
  type: project
  originSessionId: 50bdb789-2585-4258-8239-368477967e7e
  modified: 2026-10-11T09:16:02.092Z
---

**Epic BOLA-337** (สร้าง 2026-10-11) ชุดการ์ด 4 ใบ ร่างเต็มอยู่ที่ `docs/cards/drafts/bola/BOLA-follower-idp.md` (commit e55aac17 บน fix/AI-49-safe-worker-activation)

| ใบ | ชื่อย่อ | blocked-by |
|---|---|---|
| BOLA-338 | ตั้งค่า IdP ต่อ OA + ผูกตัวตนผ่าน LIFF `flow=login` (ตาราง `follower_idp_configs`, `follower_identity_links`) | BOLA-328 |
| BOLA-339 | สิทธิ์สองแหล่ง (`follower_role_assignments.source = idp_claim | admin`) → sync custom field สงวน `idp_*` → rich menu rule เดิม | BOLA-338, BOLA-335 |
| BOLA-340 | LIFF destination มี `require_verified` + `required_roles`; shell ส่ง LINE id_token ถามก่อน handoff | BOLA-339 |
| BOLA-341 | chatbot: บังคับ login, กรอง KB ด้วย `allowed_roles`, แนบ `X-Asker-Context` (JWT RS256 ของ BOLA) ไปกับ outbound webhook ต่อ OA ให้ AI platform | BOLA-339, AI-131 ฝั่ง platform |

**การตัดสินใจที่ผู้ใช้เคาะ (2026-10-11):**
- สิทธิ์ของผู้ติดตามรองรับ **ทั้งสองแหล่ง** — claim จาก IdP ผ่าน mapping ต่อ OA และแอดมินให้รายคน · union กัน · มีผลเฉพาะเมื่อ identity link ยัง verified · แอดมินถอด claim-role ไม่ได้ (กัน 3 แหล่ง)
- **ใช้ `src/repository/auth_provider/oidc.go` ร่วมกับ OIDC ของแอดมิน แต่ config แยกต่อ OA ใน DB** ไม่ผูกกับ `AUTH_MODE` เพราะ scope ต่างกัน (instance vs OA), SaaS แอดมินใช้ Kratos ไม่ใช่ OIDC, และต้องเป็นคนละ OAuth client ผู้ใช้บอกว่า "ใช้ร่วมได้ก็ดี แต่จะงง ๆ" → แก้ด้วยการวาง UI ไว้ข้าง LINE Login channel ของ OA

**ค่าที่ตัดสินแทน PO ([A-IDP-n] ในการ์ด):** link TTL 30 วัน · หนึ่ง IdP subject ต่อหนึ่ง LINE ต่อ OA (relink = revoke เก่า) · config ปิด → link เดิม `verified:false` reason `idp_disabled` · custom field `idp_role_<key>=true` ต่อ role เพื่อใช้ operator `exists` เดิม · mapping ใหม่คำนวณย้อนหลังจาก claims_snapshot ทันที · expiry job in-process ก่อน · envelope ไม่มี line_user_id ดิบ

**ข้อเท็จจริงจากโค้ดที่ใช้ตั้งการ์ด:** BOLA มี OIDC สำหรับแอดมินเท่านั้น (`AUTH_MODE=oidc`, env ระดับ instance ที่ `cmd/bola_server/helper.go:632`) · `/contacts/link` และ `uid-captured` ไม่ยืนยันตัวตน · rich menu rule engine รับ `custom_field key` + `eq/ne/contains/exists/not_exists` · LIFF shell เลือกปลายทางจาก `(line_oa_id, flow)` เท่านั้น · chatbot เดิมไม่ส่ง identity เข้า prompt · rag-core มี ActorContext employee/customer + 3 Milvus collection ตาม visibility อยู่แล้ว (`backend/rag-core/.claude/knowledge/databases.md`)

**How to apply:** งาน follower-identity / follower-permission ใน BOLA ให้เริ่มจาก BOLA-337 ไม่สร้าง epic ซ้ำ · อย่าเอา follower IdP ไปผูกกับ AUTH_MODE ของแอดมิน · คำถามเปิด Q-E1 (IdP ตัวแรก), Q-E2 (TTL), Q-E3 (endpoint ฝั่ง platform) ยังไม่มีคำตอบ ดูประกอบ [[project_bola_ai_chatbot_personalization]], [[project_bola_rbac_keto_direct]], [[reference_bola_liff_shell_error_ladder]]

**รอบรีวิว 2026-10-12 (`/po-team` รีวิวซ้ำ 4 มุม → `docs/cards/drafts/bola/REVIEW-r1-*.md`, synthesis commit bc174ebf):** ยังไม่พร้อมเข้า sprint ต้องเปลี่ยนการออกแบบ 4 จุด (สิทธิ์รั่วข้าม OA ผ่าน contact_profile · login-CSRF ยึดบัญชี · callback กลับเข้า LIFF ไม่ได้ · 341 ขัด plan AI platform ที่ freeze สมอง BOLA และให้ chat-core เซ็น asker context) · dependency จริงของ 338 คือ BOLA-333 (อยู่แค่ develop) ไม่ใช่ 328 · BOLA ไม่มีตัวตรวจ LINE id_token
**ผู้ใช้เคาะ 2026-10-12:** (1) **ไม่เก็บสิทธิ์ลง custom field** — rich menu/LIFF/chatbot ถาม `ResolveFollowerEffectiveRoles` ตรง ๆ, role catalog ระดับ workspace, mapping ต่อ OA, ตัด BOLA-335/segment ออกจากเฟสนี้ (2) **IdP ตัวแรกของลูกค้า = Google** → ต้องใช้ external browser (Google ปฏิเสธ OAuth ใน webview), ไม่มี groups claim → verified ด้วย `hd` + `email_verified`, role ละเอียดให้แอดมินกำหนด, offboarding ผ่าน refresh_token ล้ม
