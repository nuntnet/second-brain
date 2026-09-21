---
name: project_oc2plus_onpack_qr_export
description: PO เคาะ 2026-09-21 — ฟีเจอร์ export QR จาก code lot สองแบบ (รหัสเปล่า / URL) สำหรับ on-pack promo ใต้ฝา-ใต้กระป๋อง-mini card; ไม่ใช่ QR ท้ายใบเสร็จ จึงไม่ต้องรอ OC-4335/4413
metadata: 
  node_type: memory
  type: project
  originSessionId: 5112978a-fc9c-4166-a080-d49d710ed1e1
  modified: 2026-09-21T16:56:42.744Z
---

โจทย์ที่ PO สั่งเอง 2026-09-21 (ยังไม่มีการ์ดใน Jira — เพิ่งวิเคราะห์ ยังไม่เขียน)

## สิ่งที่ต้องการ

Export QR จาก code lot ที่มีอยู่แล้ว **สองแบบ พร้อม UI ที่อธิบายว่ากำลังเลือกอะไร**:

| แบบ | ขนาดจริง | เหมาะกับ | ข้อจำกัด |
|---|---|---|---|
| **QR รหัสเปล่า** `ABCD1234` | alphanumeric → **v1 = 21×21** | ใต้ฝา ใต้ซอง พื้นที่เล็ก | 🔴 กล้องปกติสแกนได้แค่ข้อความ **ต้องมีตัวอ่านในแอป = OC-4530** |
| **QR ลิงก์** `https://member.oc2.plus/<slug>/codes?c=…` (~45 ตัว) | byte → **v4 = 33×33** | ใต้กระป๋อง, mini card ในกล่องพัสดุ | ต้องมี prefill ก่อน |

ลด v4→v3 (29×29) ได้ด้วยการใช้ **path แทน query + เข้ารหัสตัวพิมพ์ใหญ่ทั้งสตริง** — ชุดอักขระ
alphanumeric ของ QR มี `/ . : -` แต่**ไม่มี `?` `=`** · แลกกับต้อง match path แบบ case-insensitive
⚠️ slug merchant แก้เองได้ (OC-4560) → slug ยาว = QR ใหญ่ขึ้นทุกดวงที่พิมพ์ไปแล้ว

## 🔑 ทำไมไม่ใช่ OC-4335 และไม่ต้องรอมัน

on-pack QR **ไม่มี order ไม่มียอดซื้อ ไม่มี `order_ref`** → **ไม่แตะ Award Engine (OC-4413),
ไม่ต้อง order-lookup ฝั่ง Patona (ใบที่ยังไม่มีเจ้าภาพ), ไม่ต้องมีตาราง `receipt_codes`**

มันคือ "โหมด A (fixed) ที่ reuse code-gen เดิม" ตามที่ OC-4335 เขียนไว้เอง ต่างกันที่**ช่องทางแจก
เป็น export ให้ไปแปะเอง ไม่ใช่ open API ยิงให้ POS พิมพ์ท้ายใบเสร็จ** (PO ระบุความต่างข้อนี้เอง)

🟡 **OC-4422 ชื่อ "Preload Code Lot + CSV Export/Import" ทับโดยชื่อ** — การ์ดใหม่ต้องเขียนขอบเขต
ให้ชัดว่า = *render QR จาก lot ที่มีอยู่* ไม่ใช่ preload/async-bind ของ OC-4422 ไม่งั้นโดนปิดเป็นซ้ำ
หรือถูกดูดไปรอ OC-4335 ที่ blocked อยู่ · OC-4530 เป็น**ผู้บริโภค**ของ QR แบบรหัสเปล่า ไม่ใช่ blocker ทั้งใบ

## แผนแตกการ์ดที่เสนอ (4 ใบ)

| # | ใบ | รีโป | ติด |
|---|---|---|---|
| **A** | member FE รับ `?c=` → prefill + auto-inquiry | member FE ล้วน | **ไม่ติดอะไร — ต้องมาก่อนทุกใบ** |
| **B1** | เพิ่มคอลัมน์ `claim_url` ใน XLSX เดิม | exporter นอกเวิร์กสเปซ | หา repo ก่อน |
| **C** | QR รูปเดียวของ template ชนิด `single` | backoffice FE ล้วน | รอ A |
| **B2** | ZIP ของ SVG+PNG + `manifest.csv` สองแบบ | exporter นอกเวิร์กสเปซ | รอ A, B1 |

ถ้ายังไม่มี A → `claim_url` ไม่มีอะไรให้ใส่ และ QR ห่อลิงก์ตาย

## ข้อสังเกตที่เปลี่ยนรูปงาน

**โรงพิมพ์ใหญ่ไม่อยากได้รูป QR** — งานใต้ฝาคือ variable data printing เขารับ CSV/XLSX แล้ว
generate QR เองใน RIP → **XLSX ที่มีอยู่เกือบพอแล้ว ขาดแค่คอลัมน์ `claim_url` (= ใบ B1 ราคาถูกมาก)**
ไฟล์รูปจริงมีค่ากับงานเล็ก/พิมพ์เอง · lot ใหญ่สุด 100,000 → ZIP แสนไฟล์ไม่ควรสร้าง ต้องมี cap

รายละเอียดกลไก code lot + exporter อยู่ที่
[[reference_oc2plus_code_lot_lifecycle_and_external_exporter]] · บริบท OC-4335 เดิมที่
[[project_loyalty_point_cluster]] · ฝั่งสแกน [[project_oc4530_counter_scan_reanalysis]]
