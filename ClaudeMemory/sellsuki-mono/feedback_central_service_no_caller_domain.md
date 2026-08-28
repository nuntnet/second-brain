---
name: feedback_central_service_no_caller_domain
description: service กลางห้ามรู้ domain ของผู้เรียก — ความหมาย/การบังคับ scope เป็นของ service ที่กิน credential นั้น
metadata:
  type: feedback
---

2026-08-28 ผมเสนอให้ sellsuki-keyring (service API key กลาง) มีทะเบียน scope เพื่อกัน scope drift user ท้วงทันทีว่า
**"ความหมายของ scope กับการบังคับ fail-closed น่าจะเป็นของ service ที่ใช้ api key นั้นเป็นผู้กำหนด ไม่งั้นจะกลายเป็น keyring ไปรู้ domain ของ caller"** — และถูก

**Why:** ถ้า service กลางต้องรู้ว่า `campaign.redeem.event` แปลว่าอะไร มันก็ต้องรู้ domain ของผู้เรียก ⇒ วันที่ทีมหนึ่งเพิ่ม
capability ใหม่ ต้องไปแก้ service กลางด้วย = คอขวดของทุกทีม และทำลายเหตุผลที่มันเป็นของกลางได้ตั้งแต่แรก
(ตรงกับ D-98 ของ keyring และกับเส้นแบ่งที่ทีม OC2Plus เคาะเองไว้ว่า "central เก็บ scope เป็น opaque ห้ามมี enum")

**How to apply:** เวลาออกแบบ service กลาง (key/identity/config/quota) แยกให้ชัดระหว่าง
- **รูปแบบ** (format, namespace, ใครเป็นเจ้าของคำนำหน้า, ทะเบียนว่ามี app/kind อะไรบ้าง) → ของกลางบังคับได้ ไม่ต้องรู้ความหมาย
- **ความหมาย + การบังคับ** (scope นี้ปลดล็อกอะไร, endpoint ไหนต้องใช้) → ของ service ผู้เรียกเสมอ

ถ้าข้อเสนอทำให้ของกลางต้องรู้ความหมาย = เสนอผิด ให้ลดรูปเหลือกฎ format แทน

ดู [[project_sellsuki_keyring_vs_oc2275]] · [[project_quota_not_feature_gate]]
