---
name: feedback_mock_mode_ui_review_check_real_adapter_before_calling_defect
description: รีวิว UI จาก mock build ผิด 3/3 ข้อที่ขึ้นการ์ด — enum ดิบ, ชื่อลูกค้า •••0002, ข้อความไทยใน EN เป็นของ mock ทั้งหมด · ก่อนเรียกว่า defect ให้เทียบ real adapter
metadata:
  type: feedback
---

2026-10-23 รีวิว UX ของ AI Chat console ด้วย `web-ai-chat-admin-mock` (VITE_USE_MOCKS=true) แล้วรายงานให้ผู้ใช้ 3 ข้อที่ต่อมา researcher พิสูจน์ว่าเป็นของ mock เท่านั้น:
- `low_confidence` ดิบใน banner: มีแต่ `mockConversationAdapter.ts` ที่ตั้ง `escalationReason`, adapter จริงไม่ตั้ง → บน staging banner ไม่เคยขึ้น (ช่องว่างจริงคือ thread ไม่บอกเหตุผลเลย ทั้งที่มี `triage.reason`)
- ชื่อ "Facebook Messenger •••0002": เป็น fallback ของชื่อว่าง; ท่อชื่อลูกค้าสร้างครบแล้ว (migration 0106 → DTO → adapter)
- ข้อความเตือนภาษาไทยใน UI อังกฤษหน้า Overview: เป็น string ใน `mockDashboardAdapter.ts:63`; ของจริงเป็นอังกฤษและไม่แปล

**Why:** mock fixture เขียนโดยคนเดียวกับ UI จึงเห็นของที่ production ไม่มี และไม่เห็นของที่ production มี (เหมือน [[feedback_verify_as_the_user_sees_it]]) ผมรายงานผู้ใช้ไปแล้วก่อนตรวจ

**How to apply:** ก่อนเขียนว่าหน้าจอ mock "แสดงค่าดิบ/ข้อมูลผิด" ให้ grep ว่า field นั้นมาจาก adapter จริงหรือเฉพาะ mock (`packages/adapters/src/chatCore*Adapter.ts` เทียบ `mock*Adapter.ts`) และเช็ก DTO ฝั่ง chat-core แล้วใส่ label "mock-mode only" ในรายงานจนกว่าจะเห็นบน staging · วัด layout/contrast จาก mock ได้ แต่ข้อความและค่าข้อมูลต้องยืนยันกับของจริง
