---
name: feedback_tests_derive_from_the_card_not_the_code
description: กฎเขียนเทส — การ์ด/business requirement คือ SSOT ไม่ใช่โค้ด · เน้นครบตามธุรกิจ ไม่เน้นให้ผ่าน · ห้ามแก้ expected ให้ตรงโค้ด
metadata:
  type: feedback
---

เทสต้องเขียนจาก AC และ business requirement ของการ์ด (การ์ดคือ SSOT) ไม่ใช่จากการอ่านโค้ด และเป้าหมายคือ "ครบตามธุรกิจจริง" ไม่ใช่ "ให้เทสผ่าน"

**Why:** ผู้ใช้ย้ำ (2026-10-03) หลังเห็น QA suite รันบน local ได้ 72/449 — เทสที่เขียนตามโค้ดพิสูจน์แค่ว่าโค้ดตรงกับตัวเอง · เป็นหลักเดียวกับ [[feedback_search_before_declaring_gap]] และ rules/verify-at-the-boundary

**How to apply:** กฎอยู่ใน `.claude/rules/feature-dod.md` (หัว 🧪) และ repo QA `.claude/rules/tests-from-the-card.md` (branch `refactor/local-run-variables` ยังไม่ push) — ทุก TC ระบุ AC ที่ครอบ · ตาราง AC→TC→ผล ใน ledger · AC ที่ไม่มีเทส = ช่องว่างที่ต้องบอก · เทสล้มเพราะผลิตภัณฑ์ไม่ตรงการ์ด = finding ห้ามแก้ expected · รันไม่ได้ = รายงานว่า NOT RUN · แยกผลเป็น ผ่าน/ล้มที่ผลิตภัณฑ์/ล้มที่ environment/ไม่ได้รัน · AC กำกวม = ถามคำถามเดียว ไม่เดาจากโค้ด
