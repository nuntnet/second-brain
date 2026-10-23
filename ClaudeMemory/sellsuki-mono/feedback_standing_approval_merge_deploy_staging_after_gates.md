---
name: feedback_standing_approval_merge_deploy_staging_after_gates
description: "2026-10-23 PO ให้ standing approval: การ์ดที่ผ่านเกตแล้ว (tier-A ทุกข้อมี verifier verdict + ship-check เขียว + review bot ไม่มี MUST FIX + pipeline เขียว) ให้ merge → deploy staging → รัน migration Job บน staging ได้เองโดยไม่ต้องถาม แล้วรายงานหลังทำ · PO คุมจุดเดียวคือ main → production · ยังต้องถามสำหรับ: production, ลบข้อมูล, ข้อความถึงลูกค้า, branch ของคนอื่น, Vault/secret"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-23T09:51:40.740Z
---

PO (nunt) สั่งใน session 2026-10-23 หลังรายงานสถานะโปรเจกต์ว่า "ทุกการ merge/deploy รอคุณ 1 รอบ" เป็นคอขวดที่ใหญ่ที่สุด: **"ให้ standing approval ตามข้อ 1 เลย"**

**เงื่อนไขที่ต้องครบก่อนใช้สิทธิ์นี้ (ทุกข้อ):**
1. ทุก AC tier A ใน ledger มี verdict จาก verifier อิสระ (PASS / PASS+NOTES) และ tier B มี dev checklist
2. `make ship-check` เขียว, pipeline ของ MR เขียว (job allow_failure เช่น sonar ไม่นับ)
3. review bot ไม่มี MUST FIX ค้าง (SHOULD FIX แก้หรือ dismiss พร้อมเหตุผลใน MR แล้ว)
4. ลำดับ producer → consumer และ migration-ก่อน-image ตามที่ ledger/MR ระบุ

**ทำได้เองเมื่อครบ:** merge MR เข้า develop/main ตาม git flow ของ repo นั้น · กด deploy job ของ staging · apply migration Job บน staging (manifest ใน `docs/release/jobs/`, บันทึกใน `docs/release/environments.yaml`) · แล้ว**รายงานหลังทำ** (MR/sha/job/ผล) + คอมเมนต์ Jira

**ยังต้องถามทุกครั้ง:** production, การลบข้อมูล/ปิดห้องลูกค้าจริง, ข้อความถึงลูกค้า, branch/checkout ของคนอื่น, Vault/secret/token (ผมไม่มีสิทธิ์อยู่แล้ว), การ์ดที่ยังมี MUST FIX หรือ verdict FAIL

**Why:** 2026-10-22 ผมถามอนุมัติ 5–6 ครั้งต่อวัน รอรอบละหลักชั่วโมง ขณะที่ In Review ค้าง 95 ใบ และ Done ใน 14 วัน = 0 · ความเสี่ยงที่ PO รับ: ของขึ้น staging เร็วขึ้น แต่ main→prod ยังเป็นของ PO

**How to apply:** เมื่อเกตครบ ให้เดินต่อทันทีโดยไม่ต้องรอคำว่า "merge ได้" — ใส่หัวข้อ "ทำไปแล้วภายใต้ standing approval" ในรายงาน · ถ้าข้อใดไม่ครบ (เช่น verifier ยังไม่ได้รัน) ห้ามใช้สิทธิ์นี้ลัด · เกี่ยวข้อง: [[feedback_comment_jira_card_when_loop_closes]], [[feedback_git_flow_main_develop_main]], [[reference_bola_jira_has_no_in_review_status_and_auto_mode_denies_merge]]
