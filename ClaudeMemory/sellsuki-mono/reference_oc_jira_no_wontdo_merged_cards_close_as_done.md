---
name: reference-oc-jira-no-wontdo-merged-cards-close-as-done
description: "โปรเจกต์ OC ไม่มี transition Won't Do/Duplicate (มีแค่ To Do, In Progress, Done, Blocked, Ready to test, code review) — การ์ดที่รวมแล้วต้องปิดเป็น Done โดยติด prefix บอก"
metadata:
  node_type: memory
  type: reference
  originSessionId: 53f9c3a5-efcb-4605-8af2-6e152327b40b
  modified: 2026-09-30T16:19:08.002Z
---

`getTransitionsForJiraIssue` บน OC-* (2026-09-30) คืนแค่: To Do(11), In Progress(21), Done(31), Blocked(41), Ready to test staging(51), code review(61), Ready to test DEV(71) — **ไม่มี Won't Do / Duplicate / Cancelled**

**How to apply:** ตอนรวมการ์ด ปิดใบที่ถูกรวมเป็น Done (id 31) แต่ทำก่อน 3 อย่าง: (1) comment ชี้ไปใบหลัก + บอกว่า "ปิดเพื่อไม่ให้ทำซ้ำ ไม่ได้แปลว่าส่งมอบ" (2) แก้ summary ขึ้นต้น `[รวมแล้ว → OC-xxxx]` เพราะ Done เฉย ๆ ดูเหมือนส่งมอบแล้ว (3) บอกผู้ใช้ว่าต้องปิดเป็น Done เพราะไม่มีสถานะยกเลิก · ลิงก์เก่าลบไม่ได้ (ดู [[reference_jira_editissue_adf_breakage]]) · ตัวอย่าง: [[project_oc4469_oc4428_card_merge]]
