---
name: reference_real_conversation_scenario_harness_lead_goal_step_2026_10_24
description: "2026-10-24 พิสูจน์สาย Lead / Conversation Goal / Step progress บน local ด้วยบทสนทนาจริงผ่าน docs/cards/evidence/LEAD-GOAL-STEP-2026-10-24/scenario.py (ลูกค้าคุยทีละเทิร์น อ่าน facts/step/goal/lead/session ต่อเทิร์น) · ผลคือ 'โค้ดมีแต่วงไม่ปิด': ไม่มีอะไรสร้าง lead จากแชท, slot number/boolean/date เติมไม่ได้, converted 9/10 บนข้อความไม่เกี่ยว, placeholder 'CTA only when prompted' หลุดถึงลูกค้า · ต้องรันซ้ำบน staging ที่มี KB เพราะ local ไม่มี KB"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-24T01:29:43.053Z
---

ใช้ `scenario.py <workspace_id> <peer-suffix> "<เทิร์น1>" "<เทิร์น2>" …` (เทิร์นที่มี ` || ` = หลาย bubble ห่าง 1 วิ) — ยิง `POST /internal/workspaces/{ws}/messages` (token local-dev-token) รอ `inbound_ai_work` settle แล้วพิมพ์: คำตอบเต็ม AI, `customer_facts`, `checkpoint_snapshots`, `checkpoint_step_changes`, goal states/occurrences, ตาราง lead, `human_mode`/fallback. workspace ที่ครบ playbook+schema+goal บน local = `2d78a4b0-7390-4801-b985-9e8d5e006938` (Test Insurance, company 85fa00dd; identity ทดสอบได้ role 17 บน company นี้แล้วบน rps local)

**กับดักตอนอ่านผล:** `psql left(body,160)` ตัดข้อความ → ข้อความ placeholder ที่หลุดท้ายข้อความไม่เห็น (verifier จับได้ ผมเห็นตอนนับ `like '%CTA only when prompted%'`) · สรุปตัวเลขของ verifier ผิดได้ (บอก 57 AC จริง 51) → นับจากตารางเอง · local ไม่มี KB ทำให้ "ไม่มีข้อมูล…เชื่อมต่อเจ้าหน้าที่" บางส่วนเป็น artifact · guardrail ตัวเลข (`containsUngroundedNumericClaim`) ถูกข้ามเมื่อมี KB grounding จึงต้องทดสอบซ้ำบน staging

ผลและร่างการ์ด: `docs/cards/evidence/LEAD-GOAL-STEP-2026-10-24/ANALYSIS.md`, `docs/cards/drafts/LEAD-GOAL-STEP-follow-ups-2026-10-24.md` · Decision Inbox D-016/017/018 · เกี่ยวข้อง: [[reference_local_ai_chat_console_open_on_localhost_with_seed_identity]], [[feedback_verify_as_the_user_sees_it]], [[feedback_no_tiny_cards_bundle_as_ac]]
