---
name: reference_local_crm_schema_lags_repo530
description: local oc2plus_crm ตามไม่ทัน repo 530 — db-migrate record หยุดที่ 2026-09-01 และ branch develop ของ repo 530 ขาดไฟล์ที่อยู่ใน main ไป 41 ไฟล์
metadata:
  type: reference
---

เจอ 2026-10-04 ตอนเทส OC-3559: 3rdparty purchase-award ตอบ 500 บน local เพราะตารางหาย (`member_sales_activity`, `campaign_condition_purchase_product`, `campaign_reward_purchase_point`, `award_member_usage`, คอลัมน์ `member.onboarding_state`, `campaign.segment_ids`)

- ตาราง `migrations` (db-migrate) ใน DB local หยุดที่ `20260901120000` — ไฟล์หลังจากนั้นที่มีอยู่ถูก apply ด้วยมือบางส่วน (tier, point_claim, consent) บางส่วนไม่เคย
- repo 530 (`.cache/oc2plus-line-crm-migration`) **`develop` มีไฟล์หลัง 2026-09-01 แค่ 7 ไฟล์ ส่วน `main` มี 48** → "รันตาม develop" ไม่พอ ต้องดู `main`
- วันนั้น apply ลง local เฉพาะที่ purchase-award ต้องใช้ (member-api 037, 039 + repo 530 `20260911000000`, `…000100`, `…012000`, `20260918000000`, `20261002120000`) — ที่เหลือยังไม่ได้เช็ค
- เจอ 500 บน local ที่ log บอก `relation … does not exist` / `column … does not exist` → เป็น schema ค้าง ไม่ใช่บั๊ก; เช็คด้วย `to_regclass` ตาม [[reference_migration_files_are_not_applied_schema]]
