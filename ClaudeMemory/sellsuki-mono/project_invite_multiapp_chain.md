---
name: project_invite_multiapp_chain
description: "Invite→เข้า app chain (PAT-2542): PAT-2553 return_to ในอีเมล verify, PAT-2554 deep-link 3 apps, OC-4371 OC2Plus bridge; BOLA ใช้ Kratos cookie ไม่ต้อง bridge"
metadata: 
  node_type: memory
  type: project
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

Chain "invite link → สมัคร/verify → เข้า company/store ของ app ที่ถูกเชิญ" (audit 2026-07-10, epic PAT-2542):

**Gap ที่พบจากโค้ด:** `return_to` ร้อยครบ login↔registration↔notice แต่**ขาดตรง verification email** — `CreateNativeVerificationFlow` (kratos-ui-go `verification_controller.go`) เป็น native flow ไม่พก return_to; หลัง accept invite ไป `/invite/myApps` ให้เลือก app เอง โดย `handler.ts` มี `APP_LINKS.oc2plus = '/'` (placeholder) และไม่มีลิงก์ BOLA

**การ์ดที่เปิด (2026-07-10):**
- PAT-2553 — kratos-ui-go เปลี่ยนเป็น browser verification flow + return_to + allowlist (blocks PAT-2554)
- PAT-2554 — invitation SPA: app destination registry + auto-redirect single-app + deep-link พร้อม company/store context (blocked by PAT-2553, OC-4371)
- OC-4371 (OC board ตาม [[feedback_oc_pat_board_ownership_rule]]) — bridge แลก Kratos session → backoffice session_key/JWT
- PAT-2543 แก้แล้ว: TIER 3 → multi-app registry (AC13) + AC14 invite path bypass /welcome; ภาพ inline เดิมถูกถอดเป็น note ชี้ Attachments (กัน render พัง)

**ข้อเท็จจริง auth ต่อ app:** BOLA SaaS = `VITE_AUTH_MODE=kratos` ใช้ cookie เดียวกัน redirect ตรงได้ ([[project_bola_auth_mode_deployment]]); OC2Plus backoffice = session_key→storeJWT/userJWT (`services/session/index.ts`) คนละชั้นกับ Kratos จึงต้องมี bridge; Patona = Seller Center + 3-tier ของ PAT-2543
