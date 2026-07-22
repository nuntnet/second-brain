---
name: bola-reply-token-epic
description: "BOLA-294 reply-token epic — 5-card plan, decisions เคาะแล้ว, latent group-fallback bug ที่ webhook.go:533-535"
metadata: 
  node_type: memory
  type: project
  originSessionId: c3e9cee3-ba99-426a-95d7-d9dc7b77441a
  modified: 2026-07-22T17:52:47.651Z
---

Epic **BOLA-294** "[Reply Token] ตอบแชท 1:1 & Group ด้วย LINE Reply API โดยไม่เสีย message quota" (สร้าง 2026-07-23) — reply message ไม่นับ quota (official pricing doc); push เข้า group นับตามจำนวนสมาชิก; reply token one-time, อายุ ~30–60s, สูงสุด 5 messages/ครั้ง

สถานะ code ตอนวางแผน: auto-reply เป็น reply-first + push fallback อยู่แล้ว (`webhook.go:527-535`); AI chatbot + chat inbox (SendHumanMessage) ยัง push ล้วน; reply template = snippet library ไม่ส่งเอง; replyToken ไม่ถูก persist

**Latent bug ที่เจอตอน prioritize:** fallback ที่ `webhook.go:533-535` push ไป `event.Source.UserID` ซึ่งผิดสำหรับ group/room event (ไม่เข้ากลุ่ม/no-op) — แก้ใน Card B

แผน 5 ใบใต้ epic: A (AI chatbot reply-first + วาง shared send-outcome emitter) ∥ C (persist token+expiry บน chat session) → B (group fallback fix, หลัง A กัน conflict) → D (inbox/template opportunistic reply, blocked-by C) → E (metric reply vs push_fallback, blocked-by A)

**Decisions ที่ user เคาะแล้ว (อย่า re-litigate):** emitter อยู่ Card A; Card E = count เท่านั้น ไม่คำนวณ quota-saved ของ group (follow-up); ตอบเกิน 5 objects = reply 5 แรก + push ส่วนเกิน + log; Card C เป็น enabler เขียน narrative ผูกผล + blocked-by D

**Why:** future sessions ที่หยิบ card เหล่านี้ไป dev ต้องรู้ decision + bug context โดยไม่ต้อง re-research
**How to apply:** ก่อนแก้ send path ใน bola-backend เช็ค card ใต้ BOLA-294 ก่อน; pattern reply-first ให้ copy จาก `webhook.go:527-535` แต่ระวัง fallback target ของ group. เกี่ยวข้อง: [[project_bola_contact_profile_model]], [[project_bola_workspace_scoping_bugs]]
