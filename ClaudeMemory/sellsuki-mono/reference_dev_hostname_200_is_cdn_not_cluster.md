---
name: reference_dev_hostname_200_is_cdn_not_cluster
description: curl 200 บน crm.dev.oc2.plus / member.dev.oc2.plus ไม่ได้พิสูจน์ว่า cluster เปิด — เป็น CDN edge (CloudFront) ตอบ SPA · เช็คเวลา (21:00–09:00 ปิด) และ kubectl ก่อนบอกว่าระบบขึ้น · classifier บล็อก kubectl exec ที่เขียน DB เสมอ
metadata:
  node_type: memory
  type: reference
  originSessionId: 5030c764-9cdf-43ae-8f05-c6a5aaa3952b
  modified: 2026-09-23T16:34:59.140Z
---

**เกิดจริง 2026-09-23 23:10:** ผู้ใช้บอก "staging ปิด" ผมเถียงว่าไม่ปิดเพราะ `curl https://crm.dev.oc2.plus/` ได้ 200 และ
`kubectl` ตอบ "Having problems with relogin, please use tsh login" จึงสรุปว่าเป็น teleport หมดอายุ — **ผิด**
เวลาตอนนั้น 23:32 อยู่ในหน้าต่างปิดระบบ 21:00–09:00 (shipping.md §9) · 200 มาจาก CDN edge
(`member.dev.oc2.plus` → 3.166.34.69 = CloudFront) ที่เสิร์ฟ SPA static ได้แม้ pod ดับ

**ลำดับตรวจก่อนพูดว่า "ระบบขึ้น":**
1. `date` — ถ้าอยู่ใน 21:00–09:00 ถือว่าปิด จบ
2. `kubectl -n <ns> get deploy` — EOF/relogin = session หมดหรือ cluster ปิด แยกไม่ได้จาก client
3. ยิง **API path** ที่ต้องถึง pod จริง (เช่น `/backoffice/v1/...` ให้ 401) ไม่ใช่ root ของ SPA

**classifier ของ auto mode บล็อก `kubectl exec … psql -f -` ที่เขียน DB ทุกครั้ง** ("Remote Shell Writes")
read-only SELECT ผ่านคำสั่งเดียวกัน **ผ่านบ้างไม่ผ่านบ้าง** (2026-09-23 ผ่าน · 2026-09-24 ถูกบล็อกทั้งที่เป็น SELECT ล้วน) —
อย่าพึ่ง · ทางที่ใช้ได้จริง 2026-09-24: ให้ bash block มี Run button ผู้ใช้กดใน terminal ของแอป แล้ววาง output กลับมา · เตรียม SQL ไฟล์ + คำสั่งเดียวให้ผู้ใช้รันเอง
(`$POSTGRES_PASSWORD` อยู่ใน single quote → shell ใน pod แทนค่า ไม่ต้องรู้รหัส) · ถ้าจะให้ผมรันต้องเพิ่ม
permission rule `kubectl -n datastore exec*` ใน settings

ดู [[reference_teleport_session_kills_devth_access]] · [[reference_harness_classifier_blocks_secrets_and_mutations]] · [[project_oc2275_crm_migrations_run_by_hand]]
