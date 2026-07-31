---
name: reference_messaging_backend
description: "Central messaging service = sellsuki-messaging-backend (active; OTP/SMS via Thaibulk, extensible provider abstraction) — email lives here (OC-4343); sellsuki-service-messaging is DEAD"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-07-31T10:26:50.836Z
---

**Central messaging service = `gitlab.sellsuki.com:sellsuki/sellsuki/backend/sellsuki-messaging-backend`** (not a monorepo submodule).

- Active (commits เม.ย. 2026, vault + staging-th pipeline) · Go, generics_server layout เดียวกับ sellsuki-service-consent · มี AGENTS.md + README ละเอียด
- ทำหน้าที่ abstraction layer ระหว่าง app ↔ messaging providers: วันนี้มี OTP/SMS ผ่าน **Thaibulk** · multi-tenant provider config (`message_provider_config_repository`, `message_action_repository`) · activity log + credit tracking · Kratos identity/permission · distributed lock
- Extension pattern (README สอนไว้): เพิ่ม enum ใน `src/use_case/model/message_service.go` (มี `otp`,`sms`) / `message_provider.go` (มี `thaibulk`) → เขียน repository impl ใหม่ → เพิ่ม case ใน use-case switch
- **Email service (OC-4343) ทำใน repo นี้** — service type `email` + provider (เสนอ SES) + template ในตัว (`[var=key][/var]`) · consumer แรก = OC-4215
- ⚠️ `sellsuki-service-messaging` (group เดียวกัน) = repo ตายแล้ว commit สุดท้าย ต.ค. 2023 — แต่มีของ salvage ได้: FB/LINE REST client (send/attachment/token), multicast/scheduler (ไม่มีขา webhook receive เลย)
- **Scope ขยาย (เคาะ CTO 2026-07-26, [[ai-chat-platform-plan]]):** repo นี้จะเป็น **unified message gateway** ครอบ OTP + email + SMS + **social media chat** (FB webhook receive/normalized message/conversational send) — ยุบ sellsuki-service-messaging ทิ้งหลัง port ของ; BOLA จะ forward chat webhook เข้ามาที่นี่ก่อนส่ง Chat Core; OTP/SMS path เดิมต้องไม่กระทบ
- Group path จริงคือ `sellsuki/sellsuki/backend/...` (ซ้อน sellsuki สองชั้น) — probe แค่ `sellsuki/...` จะไม่เจอ. เสริม lesson ใน [[reference_oc2plus_member_frontend]]: ls-remote ให้ถูก group
