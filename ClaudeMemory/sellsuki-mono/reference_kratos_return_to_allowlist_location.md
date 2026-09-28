---
name: reference-kratos-return-to-allowlist-location
description: allow-list return_to ของ Kratos ทุก env อยู่ที่ infra/ory-helm/<env>/kratos-values.yaml ในโมโนรีโป — เป็นสำเนา ต้องยืนยันด้วยการกดจริง
metadata:
  type: reference
---
`selfservice.allowed_return_urls` ของแต่ละ env อยู่ใน `infra/ory-helm/{development-th,staging-th,production,...}/kratos-values.yaml`
ณ 2026-09-28 มี wildcard `*.dev-th/staging-th.sellsuki.com`, `*.oc2.plus`, `*.patona.online` และ BOLA มี host แยกเฉพาะ staging/prod ส่วน BOLA dev ไม่อยู่ในลิสต์

ไฟล์นี้เป็นสำเนาใน repo ไม่ใช่ค่าที่อ่านจาก cluster ถ้า return_to โดนปฏิเสธจริงให้สงสัยว่าไฟล์กับ cluster ไม่ตรงกัน ใช้ตอนตัดสิน AC-8 ของ PAT-2735 และงาน redirect อื่นที่ผ่าน Kratos
