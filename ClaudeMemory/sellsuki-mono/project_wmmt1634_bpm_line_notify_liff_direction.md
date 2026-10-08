---
name: project_wmmt1634_bpm_line_notify_liff_direction
description: WMMT-1634 (แจ้งผู้แจ้งปัญหาฟอร์ม BPM ทาง LINE OA @bxy2141c) — ทางที่เสนอ 2026-10-02 คือผูกด้วย LIFF + ID token แทนพิมพ์รหัสในแชท แล้วใช้ Jira Automation ยิง APM ของ BOLA (target_line_user_id) · ฟอร์ม Jira business prefill ไม่ได้ · ยังไม่มีใครเคาะ
metadata:
  node_type: memory
  type: project
  originSessionId: 53f9c3a5-efcb-4605-8af2-6e152327b40b
  modified: 2026-10-08T12:50:48.416Z
---

การ์ด [WMMT-1634](https://sellsuki.atlassian.net/browse/WMMT-1634) อยู่บอร์ด Wizemoves Martech คู่กับ WMMT-1633 (ระบบอีเมลของน้องบอล) ตัวการ์ดเขียนไว้ว่าให้ลูกค้าพิมพ์รหัสผูกเรื่องในแชท แล้วใช้ webhook จับ

**ข้อเสนอที่คุยกับผู้ใช้ (2026-10-02, ยังไม่ได้แก้การ์ด):**

1. **ผูกเรื่องด้วย LIFF:** ลิงก์ `liff.line.me/{id}?ref=<รหัส>`
   - BOLA ตรวจ ID token แล้วอ่าน/เขียน Jira เอง ไม่ต้องแตะ webhook ของ OA
   - ทำเป็น extension ใน BOLA แบบ `src/extensions/lon_rgb`
   - ห้ามใช้ `/public/liff/uid-captured` เพราะไม่ยืนยันตัวตน
   - LIFF เดิมของ BOLA ใช้ endpoint ตัวเดียว แล้วเลือกปลายทางจากตาราง destination ตาม `flow` ในลิงก์ ยังไม่ได้เช็กว่า `ref` ติดไปถึงหน้าปลายทาง
2. **แจ้งเตือนเมื่อเปลี่ยนสถานะ:** Jira Automation ยิง "Send web request" ไปที่ BOLA APM `/webhook/apm/:id` (header `X-Security-Token`) แบบ dynamic target `target_line_user_id`
   - APM ตอบ 202 แบบ async จึงไม่รู้ผลการส่ง ต้องให้ BOLA คอมเมนต์กลับ Jira เองถ้าส่งไม่สำเร็จ
3. **ผู้ใช้ถามว่าถ้ากดฟอร์มจาก Rich Menu (อยู่ใน LINE แล้ว) ต้องผ่านอีเมลอีกไหม:**
   - ฟอร์ม Jira business (`jira/core/form`) เติมค่าผ่านลิงก์ไม่ได้
   - JSM portal เติมได้ แต่ใช้กับ Forms ไม่ได้ และต้องใช้โทเคนใช้ครั้งเดียวแทนการส่ง userId ตรง ๆ
   - ทางเลือกคือฟอร์มของเราเองใน LIFF หรือย้ายไป JSM

**ยังค้าง:**
- OA @bxy2141c ต่อเข้า BOLA หรือยัง
- เลือกทางไหนระหว่าง: ฟอร์มเดิม + ลิงก์ในอีเมล / ฟอร์มใน LIFF / JSM

**How to apply:** ถ้าเรื่องนี้กลับมา ให้เริ่มจากข้อค้างสองข้อนี้ อย่าออกแบบใหม่ตั้งแต่ต้น ดูประกอบ [[project_bola_apm_webhook_design]] และ [[reference_liff_single_endpoint_flow_in_link_not_endpoint]]
