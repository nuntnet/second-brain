---
name: feedback_ground_claims_file_line
description: "User ท้าทาย \"gap จริงจาก code หรอ\" — ทุก claim ในการ์ดต้องยึด file:line จริง และแยก verified vs อนุมานให้ชัด"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

User ท้าทายซ้ำสองรอบ (2026-07: "2292 จริงๆก็เชื่อไม่ได้นะ ต้องไปเช็คจาก code" · 2026-07-10: "ที่เขียนการ์ดมา มี gap จริงๆจาก code หรอ")

**Why:** การ์ด/analysis ที่เขียนจากความจำหรือ doc เก่าเคยผิด (vocab ที่แต่งขึ้นใน PAT-2292) — user ต้องการหลักฐานระดับ file:line เสมอ

**How to apply:** (1) ก่อนเขียน gap/behavior ใด ๆ ลงการ์ด ต้องอ่านโค้ด/config จริงก่อน (รวม infra config เช่น kratos.yml ใน docker mount ไม่ใช่แค่ app code) (2) อ้าง path:line ในการ์ดทุก claim หลัก (3) สิ่งที่ยังเป็นการอนุมาน/สมมติฐาน ต้อง flag ชัดในการ์ด ("ต้อง verify ตอน implement") ห้ามเขียนปนเป็น fact (4) เมื่อถูกถาม ให้ตอบเป็นตาราง claim→หลักฐาน แยกส่วน verified กับ assumption — เกี่ยวกับ [[feedback_qa_reproduce_guide]]
