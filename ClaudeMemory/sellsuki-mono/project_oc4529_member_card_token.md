---
name: project_oc4529_member_card_token
description: "OC-4529 บัตรสมาชิก QR — PO เคาะ token อายุ 5 นาที ใช้ครั้งเดียว (ขัดกับ 'รีเฟรชทุก 60 วินาที' ที่ยังค้างใน design v3 + AC), สร้างครบทั้ง 3 ฝั่งแล้ว; ฝั่งพนักงานสแกนยังไม่มีคนทำ = OC-4539"
metadata: 
  node_type: memory
  type: project
  originSessionId: fa22ebb1-f715-4667-85a1-61ebbcc816ab
  modified: 2026-09-12T23:18:24.404Z
---

**เคาะโดย PO 2026-09-13** ตอนสั่ง `/feature OC-4529`

## สัญญาที่ลงโค้ดไปแล้ว

| ฝั่ง | ของจริง |
| --- | --- |
| ออกโทเคน | `GET /v1/me/card-token` (member-api, session cookie เท่านั้น ไม่มี field รับ member_id) → `{token, expires_at}` |
| verify/consume | `POST /v2/openapi/member/card/verify` (3rdparty-api, API key + scope **`member.card.verify`**) |
| ตาราง | `member_card_token` — migration **017** ใน `member-api/migrations/` |

**token = opaque random เก็บเฉพาะ SHA-256 hash · อายุ 5 นาที · ใช้ครั้งเดียว · ผูก company_id**

consume เป็น `UPDATE ... SET consumed_at=now() WHERE token_hash=$1 AND consumed_at IS NULL
AND expires_at > now() AND company_id=$2 RETURNING member_id` — **คำสั่งเดียว** สองเครื่องสแกน
พร้อมกันจึงไม่ผ่านทั้งคู่ · ใช้ `now()` ของ DB ไม่ใช่ `time.Now()` ของ Go (กันเวลาเครื่องเพี้ยน)

ตารางใช้ **`timestamptz`** ทั้งหมด ต่างจากตารางอื่นในสกีมา CRM ที่เป็น naive `timestamp` —
ตั้งใจ เพราะอายุโทเคนแค่ 5 นาที การเลื่อนตาม host TZ (ดู
[[reference_naive_timestamp_columns_shift_by_host_tz]]) จะกลายเป็นช่องโหว่ ไม่ใช่แค่แสดงผลเพี้ยน

## 🔴 ข้อความ "รหัสรีเฟรชทุก 60 วินาที" ยังค้างอยู่ 2 ที่

design v3 (`ovQR` ใน `OC2 Plus Loyalty v3 - Tier.dc.html`) **และ AC ข้อแรกของ OC-4529**
ยังเขียน 60 วินาที ซึ่ง**ถูก superseded แล้ว** · summary ของการ์ดแก้เป็น "อายุ 5 นาที ใช้ได้
ครั้งเดียว" และมีคอมเมนต์ทับไว้ แต่ **body ของ description ยังไม่แก้** (เลี่ยง ADF ทำไทยเพี้ยน
ดู [[reference_jira_editissue_adf_breakage]]) — ใครอ่าน description ตรง ๆ จะทำผิด

## สถานะการ์ด (2026-09-13)

- 🔴 tier **ตกไปแล้ว** — `me_tier_v1.go` `GetMeTier` มีจริง และ UI โชว์ tier ได้
- 🔴 token **เคาะแล้ว** (ข้างบน)
- 🔴 **ฝั่งพนักงานสแกนยังไม่มีเจ้าของ** → เปิดการ์ด **OC-4539** (ครอบ 3 ทาง: สแกนบัตรสมาชิก /
  ออก QR ของร้าน / รับแจ้งเบอร์โทร) · **ไม่ block การสร้าง OC-4529** เพราะ POS ของลูกค้า
  ต่อผ่าน `/v2/openapi` ได้เอง — แต่ block การใช้งานจริงที่หน้าเคาน์เตอร์

การ์ดใหม่ที่เปิดรอบเดียวกัน: **OC-4540** เช็คอินประจำวัน · **OC-4541** ศูนย์การแจ้งเตือน
(ทั้งคู่มีปุ่มตายอยู่บน production จริงแล้ว) · ทั้งสามใบใต้ epic OC-4349 ไม่ใส่ sprint

⚠️ **3rdparty-api อยู่คนละ branch** กับที่เหลือ: `codex/oc-4344-web-member-identity`
ส่วน member FE / member-api / monorepo อยู่ `codex/oc-4344-customer-app-integration`

## ยังไม่ได้ทดสอบในเบราว์เซอร์

โค้ด build ผ่าน เทสเขียว migration ลง DB จริงแล้ว แต่ **service บนพอร์ต 8102 เป็นไบนารีเก่า
ที่อีก tool สั่งรันไว้** (`oc4344-me`) ไม่ rebuild ตาม route ใหม่ → `/v1/me/card-token` กับ
`/v1/session/logout` ตอบ 404 ในเครื่อง · ต้อง restart member-api ก่อนถึงจะ QA เส้นจริงได้

เชื่อม [[reference_oc2plus_member_app_local_qa_session]] [[project_oc2plus_member_app_design_v2]]
