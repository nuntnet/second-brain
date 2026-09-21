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

## 🚫 LIFF URL ใน QR — ตัดออกอย่างเป็นทางการ (เคาะ 2026-09-22)

PO ถามว่าจะทำ QR เป็น LIFF URL ด้วยได้ไหม · คำตอบคือ **อย่า** และตันสองชั้น ไม่ใช่แค่ซับซ้อน:

1. **เคยทำแล้วถอนออกโดยตั้งใจ** — OC-4511 เพิ่ม `CUSTOMER_APP_LIFF_ID` ประกอบ
   `https://liff.line.me/{id}/{slug}/{page}` แล้ว backoffice-api **MR !551 (`238fcd7`) ลบทิ้ง**
   มติ: ห้ามมี LIFF ตัวที่สองของ member app (ดู [[project_oc2plus_liff_shell_is_the_line_entry]])
2. **ตันเชิงโครงสร้าง** — shell ของ BOLA หา destination จากตาราง `liff_shell_destination
   (line_oa_id, flow)` = แถวต่อ **flow** ไม่ใช่ต่อโค้ด · และ shell **ทิ้ง query ขาเข้า**
   (`window.location.replace(destination + ?line_user_id&line_oa_id&…)` ต่อพารามิเตอร์ของตัวเอง
   ไม่ forward ของเดิม) → `?c=CODE` หายกลางทางเงียบ ๆ
   · `member-api/src/usecases/liff/entry.ts` อ่านแค่ `lid`/`liff.state`/`aid`/`module`
   จบที่ redirect/signing-in/idle — ไม่มีเส้นทางไปหน้า redeem เลย

**หลักที่ได้:** เอาของที่เปลี่ยนได้ (LIFF id / shell / flow config) ไปพิมพ์ลงของที่เปลี่ยนไม่ได้
(บรรจุภัณฑ์) = ล็อกความผิดพลาดไว้ถาวร → QR ห่อ **web URL ธรรมดา** ตัวเดียวพอ

## 🔑 อยากได้ LINE uid ทั้งที่เข้าทาง URL — backend มีครบแล้ว (พบ 2026-09-22)

OC-4523 (Ready to test DEV) `POST /company/{slug}/auth/line` ตรวจ id_token โดยส่งให้ LINE เอง
ที่ `POST /oauth2/v2.1/verify` (`member-api/src/repository/line_repository/rest.go:152`)
โดย audience = **LIFF channel id ของ OA นั้น** ดึงจาก BOLA ต่อ OA (`auth_line.go:153`, จงใจกัน
ข้ามผู้เช่า) พร้อม replay guard 1 token = 1 session, rate limit, audit

🔑 **LIFF app อยู่ใต้ LINE Login channel → LINE Login แบบเว็บ (OAuth) ใช้ channel เดียวกัน →
id_token ได้ `aud` เดียวกัน → endpoint เดิมรับได้โดยไม่ต้องแก้ backend เลย**

ที่ขาดคือ: FE ทำ OAuth dance + endpoint อ่านเล็ก ๆ บอกว่า slug นี้ใช้ OA/channel id ไหน
(FE ต้องรู้ `client_id` ถึงจะเริ่ม OAuth และ endpoint บังคับส่ง `line_oa_id`)
🔴 **+ ลงทะเบียน callback URL ในคอนโซล LINE Login ของแต่ละร้าน** — ร้านที่ไม่ได้ลงทะเบียน
พังเงียบเฉพาะร้านนั้น ⇒ ต้องเข้า company readiness (`company_readiness.go`) ไม่ใช่ env var
จึง `ship-check` มองไม่เห็น

## OC-4530 ต้องอ่านได้ทั้งสองแบบ (PO สั่ง 2026-09-22)

resolver ต้องแยก ≥5 รูปแบบ: บัตรสมาชิก (OC-4529) / คูปอง (OC-4526, 4547) / รหัสเปล่า /
**URL ของเรา → ถอด `c` แล้วอยู่ในแอป ห้ามเด้งออก browser** / อย่างอื่นหรือ slug ร้านอื่น → บอกชัด
· checksum (sum mod 36) คัดกรองในเครื่องได้ฟรีก่อนยิง network · slug เก่าต้องผ่าน (OC-4560)
· การ์ดอยู่ To Do ยังไม่เริ่ม → แก้ AC ในใบเดิมได้ แต่ **รายงานให้ PO เคาะ ไม่แก้เอง**
([[feedback_report_wrong_cards_dont_edit]])

## แผนแตกการ์ดที่เสนอ (5 ใบ + แก้ OC-4530)

| # | ใบ | รีโป | ติด |
|---|---|---|---|
| **A** | member FE รับ `?c=` → prefill + auto-inquiry | member FE ล้วน | **ไม่ติดอะไร — ต้องมาก่อนทุกใบ** |
| **A2** | LINE Login บนเว็บ → session ที่มี LINE uid | member FE + member-api (endpoint อ่าน) + readiness | A |
| **B1** | เพิ่มคอลัมน์ `claim_url` ใน XLSX เดิม | exporter นอกเวิร์กสเปซ | หา repo ก่อน |
| **C** | Preview QR + test sheet PDF (6/8/10/12/15 มม.) + lot ทดสอบ · รวม QR ของ template `single` เข้ามาด้วย | backoffice FE + backoffice-api | รอ A |
| **B2** | ZIP ของ SVG+PNG + `manifest.csv` สองแบบ | exporter นอกเวิร์กสเปซ | รอ A, B1 |

ถ้ายังไม่มี A → `claim_url` ไม่มีอะไรให้ใส่ และ QR ห่อลิงก์ตาย

## ข้อสังเกตที่เปลี่ยนรูปงาน

**โรงพิมพ์ใหญ่ไม่อยากได้รูป QR** — งานใต้ฝาคือ variable data printing เขารับ CSV/XLSX แล้ว
generate QR เองใน RIP → **XLSX ที่มีอยู่เกือบพอแล้ว ขาดแค่คอลัมน์ `claim_url` (= ใบ B1 ราคาถูกมาก)**
ไฟล์รูปจริงมีค่ากับงานเล็ก/พิมพ์เอง · lot ใหญ่สุด 100,000 → ZIP แสนไฟล์ไม่ควรสร้าง ต้องมี cap

🔴 **กับดักของ preview: สแกนทดสอบ = เผาโค้ดจริง** (status → `redeemed`) แล้วปนอยู่ใน lot
ที่จะส่งโรงพิมพ์ → มีฝาขวดใบหนึ่งที่ลูกค้าเจอ "ใช้แล้ว" โดยไม่มีใครรู้ว่าใบไหน ⇒ preview ต้องโชว์
สถานะรายใบ + มี "lot ทดสอบ 5 รหัส" แยก · ปุ่ม next มีค่าเพื่อ**ไล่หาใบที่ยังไม่ถูกเผา** ไม่ใช่เพื่อดูขนาด
เพราะรหัสยาวคงที่ทั้งเทมเพลต → **QR ทุกดวงใน lot ขนาดเท่ากันเป๊ะ**

**จอตอบไม่ได้ว่าพิมพ์แล้วสแกนติดไหม** (จอเรืองแสง contrast เต็ม vs ฝาโลหะโค้งมีเงา) ⇒ ต้องมี
**test sheet PDF** ที่มี QR ดวงเดียวกันที่ 6/8/10/12/15 มม. เอาไปพิมพ์บนวัสดุจริงแล้วไล่สแกน

รายละเอียดกลไก code lot + exporter อยู่ที่
[[reference_oc2plus_code_lot_lifecycle_and_external_exporter]] · บริบท OC-4335 เดิมที่
[[project_loyalty_point_cluster]] · ฝั่งสแกน [[project_oc4530_counter_scan_reanalysis]]
