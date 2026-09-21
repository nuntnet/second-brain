---
name: reference_oc2plus_code_lot_lifecycle_and_external_exporter
description: OC2Plus code system — code เกิดตอนสร้าง lot ไม่ใช่ตอน publish campaign; ทั้ง generator และ exporter เป็น Kafka consumer ที่ไม่มีอยู่ในมอนอรีโปเลย (ตรวจ 2026-09-21)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5112978a-fc9c-4166-a080-d49d710ed1e1
  modified: 2026-09-21T16:56:03.749Z
---

## code เกิดตอนไหน — ไม่ใช่ตอน publish campaign

`CreateCodeLot` (`backoffice-api/src/use_case/code.go:249`) รับแค่ `companyId` + `codeId`
**ไม่มีพารามิเตอร์ campaign เลย** และบรรทัดท้ายยิง `ExecuteGenerateCode` ทันที
→ **กดสร้าง lot = code เกิดเลย** จะมี campaign อยู่หรือยังไม่มีก็ได้ และ lot เดียวใช้กับหลาย campaign ได้

campaign อ้าง **code template** (ไม่ใช่ lot) ผ่าน `campaign_condition_code` · `ChangeCampaignStatus`
(`campaign.go:271`) เปลี่ยนสถานะอย่างเดียว · publish ไม่ generate อะไร ·
`validateCampaignConditionCodeItemBusinessRules` (`campaign.go:986`) แค่เช็คว่า template มีจริง

**ความเข้าใจที่ผิดและฟังดูสมเหตุสมผลมาก** (PO เข้าใจแบบนี้ 2026-09-21): "code จะไม่ถูกสร้าง
จนกว่าจะอ้างใน campaign แล้ว publish ถึงเกิด" — สำคัญเพราะถ้า export QR ที่ระดับ lot
**QR จะไม่รู้จัก campaign** ใส่ชื่อแคมเปญ/วันหมดอายุ/เงื่อนไขไม่ได้ ห่อได้แค่ตัวรหัส

## 🔴 generator กับ exporter ไม่ได้อยู่ในมอนอรีโป

ทั้งสองเป็น **Kafka producer เท่านั้น** ฝั่ง backoffice-api:

| งาน | topic (dev) | ตัวเรียก |
|---|---|---|
| gen code | `development.oc2plus.crm.code.cmd.generate.v1` | `generate_code_repository/kafka.go:116` |
| export ไฟล์ | `development.oc2plus.crm.export.cmd.export.v1` | `execute_export_file_repository/kafka.go:117` |

`grep -rl` ทั้ง `backend/` แล้ว — **ไม่มี consumer ของทั้งสอง topic อยู่ที่ไหนในเวิร์กสเปซเลย**
(`kafka_consumer_worker/workers/` มีแค่ change-campaign-status กับ point-claim-notification)
ไฟล์ออกเป็น **XLSX ไป S3** (`S3_EXPORT_BUCKET`, folder `crm-export-development`)

→ **"แค่เพิ่มรูปแบบ export" = แก้ service ที่ไม่มีในเวิร์กสเปซ** ต้องหา repo ให้เจอก่อนตีขนาดการ์ด
รูปแบบเดียวกับ [[reference_oc2plus_schema_lives_in_external_repo]]

## ชนิด template — single ทำ lot/export ไม่ได้

`code.CodeType` มีสองค่าเท่านั้น: `single` | `dynamic` (`backend/entity@v1.9.7/code/code_template.go`)

- **`single`** = รหัสเดียวใช้ร่วมกัน · `CreateCodeLot` **และ** `ExportFileCodeLot` ปฏิเสธด้วย
  `ErrSingleTemplateNotSupported` → ไม่มี lot ไม่มี export
- **`dynamic`** = สุ่มเป็นชุด · `Quantity` 1–100,000 ต่อ lot · ปุ่มดาวน์โหลดเปิดเมื่อ lot status = `success`

UI: `CodeMasterList` → `CodeMasterCreate` → `CodeeMasterPreview` (ฝัง `CodeGenerateRandom` เป็นแท็บ lot)

## ฝั่ง member — redirect_uri พารหัสข้ามการสมัครได้อยู่แล้ว

`withRedirectUri()` ส่ง `window.location.href` **เต็ม ๆ** · `safeRedirectTarget()` เก็บ query+hash
(มี spec ยืนยันที่ `utils/safeRedirect.spec.ts:38`) · Login/Register อ่าน `redirect_uri` แล้ว ·
route `redeem-code` = `skipsAuthGuard:false` → เด้งไปสมัครแล้วกลับมาเอง

**ที่ขาดชิ้นเดียวคือ `CodePageImpl.tsx:17` ไม่อ่าน query param** (`useState('')`) → ไม่มี URL แบบ
`?c=<code>` ให้ QR ห่อ · OC-4363 เขียน Out of Scope ของตัวเองไว้ว่า "QR scan เพื่อ prefill = v1 กรอกมือเท่านั้น"
