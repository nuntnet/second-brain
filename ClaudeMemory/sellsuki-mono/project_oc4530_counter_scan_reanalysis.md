---
name: project_oc4530_counter_scan_reanalysis
description: "OC-4530 วิเคราะห์ใหม่ + PO เคาะ B-lite + implement OC-4542/4539 บน 3 feature branch 2026-09-13 — QR สองทิศต่างกันที่ 'ลำดับข้อมูล'; 'แสดง QR ตามเงื่อนไขสมาชิก' นิยามไม่ได้; ของที่ขาดจริงคือ backoffice เรียก verify ไม่ได้ (คนละ auth); และ OQ1 'ทับ OC-4363 ไหม' ตอบได้จาก Out of Scope ของ OC-4363 เอง = ไม่ทับ เป็นใบต่อ"
metadata:
  node_type: memory
  type: project
---

**วิเคราะห์ตามที่ PO สั่ง 2026-09-13** (ยังไม่ได้แก้การ์ดใบไหน ตาม [[feedback_report_wrong_cards_dont_edit]])
artifact: https://claude.ai/code/artifact/41b850ab-7ad5-4203-8a2b-060404a84eea

## ข้อโต้แย้งแกน — ลำดับข้อมูล ไม่ใช่ทิศกล้อง

- **ทิศ A (ลูกค้าโชว์ ร้านสแกน)** รู้ *ใคร* ก่อน → POS มีข้อมูลบิลอยู่แล้ว → server ตัดสินได้ครบ
- **ทิศ B (ร้านโชว์ ลูกค้าสแกน)** รู้ *แคมเปญ* ก่อน แต่ตอนสร้าง QR ยังไม่รู้ว่าใครจะสแกน
  → **"ดึงแคมเปญมาแล้วแสดง QR ตามเงื่อนไขของสมาชิก" นิยามไม่ได้** (ระดับสมาชิก/โควตาต่อคน/เดือนเกิด
  เป็นคุณสมบัติของ *คน* ไม่ใช่ของ QR) · ถ้าจะให้รู้ ต้องสแกนบัตรก่อน = ทิศ A ไปแล้ว = ไม่ต้องมี QR

**QR ฝั่งร้าน = ตัวแทนของการไม่มี integration ไม่ใช่ฟีเจอร์** · ถ้า POS สร้าง QR ต่อบิลได้ แปลว่าต่อ API
ได้อยู่แล้ว → ยิง `purchase.submit` ตรงกว่า · QR มีค่าเฉพาะตอนต่อ API ไม่ได้ → ที่ถูกคือ **ท้ายใบเสร็จ
(OC-4335/4422) ไม่ใช่จอเคาน์เตอร์** และเป็นคนละ journey (ลูกค้าสแกนตอนไหนก็ได้)

## 3 โจทย์ของ PO map เป็น

| โจทย์ | จริง ๆ คือ | คำตัดสิน |
|---|---|---|
| 1. เปิดหน้าประวัติเพื่อปรับแต้มเร็ว | ระบุตัวสมาชิกล้วน | **คุ้มที่สุด ทำได้เลย** ปลายทางมีครบ |
| 2. POS โชว์ QR ของแคมเปญนั้น | ผูกแคมเปญ/ยอดซื้อ | ย้ายไป OC-4335 ตัดออกจากเคาน์เตอร์ |
| 3. แสดง QR ตามเงื่อนไขสมาชิก | ทั้งสองชั้นพร้อมกัน | เจตนาถูก สื่อผิด → เป็น **รายการสิทธิ์บนจอพนักงาน ไม่ใช่ QR** |

## สถานะโค้ดจริง (ตรวจ 2026-09-13) — ต่างจากที่ OC-4539 เขียนไว้แล้ว

**มีแล้ว:** `GET /v1/me/card-token` · `POST /v2/openapi/member/card/verify` (คืน member_id/member_code/
ชื่อ-สกุล) · `POST /company/:id/member/:member_id/point/adjust` + `AdjustPointsDialog.vue` +
route `/member/:member_code` (`frontend/oc2plus-linecrm-frontend-backoffice/src/router/index.ts:89`)
· 🔑 **`FindActiveCampaignByCodeCondition(company, member, now, templateIDs)`** ใน
`3rdparty-api/src/use_case/code_redemption_openapi.go:104` — **ระบบประเมิน eligibility ต่อสมาชิกได้อยู่แล้ว**
แต่ทำตอน redeem ไม่ใช่ตอนสร้าง QR (นี่คือของที่โจทย์ข้อ 3 ต้องการ)

**ขาดจริง (ไม่ใช่ที่การ์ดเดา):**
- 🔴 **backoffice เรียก verify ไม่ได้** — verify อยู่หลัง API key ส่วน backoffice ใช้ session cookie ·
  `grep card.verify backoffice-api` = 0 → ต้องมี proxy endpoint ฝั่ง backoffice + Keto
- 🔴 **verify = consume แล้วจบ** → 1 token = 1 action · สแกนเปิดหน้าประวัติแล้วจะปรับแต้มต่อ ต้องให้ลูกค้า
  เปิดบัตรใหม่ → **เสนอให้ verify คืน staff context อายุสั้น (ผูก member+company+พนักงาน)** แทน identity เปล่า
- ไม่มี endpoint สร้างโค้ด/QR แคมเปญเลย (OC-4335/4422 ยัง To Do)
- ไม่มีทางเข้าแบบเปิดกล้องบนจอพนักงาน

## ที่เสนอให้เคาะ

1. OC-4539 OQ1 (A/B/C) — **โจทย์ข้อ 1 ปิดคำตอบเองแล้วว่าต้องมี B** เพราะ manual adjust อยู่ใน backoffice
   ของ Sellsuki · เสนอ C แบบไม่เท่ากัน (B = ทางหลักหน้าเคาน์เตอร์, A = ร้านที่มี POS)
2. OQ2 "QR ที่เคาน์เตอร์คืออะไร" → เสนอ **ตัดทิ้งทั้งข้อ** ย้ายไปท้ายใบเสร็จ
3. verify = burn หรือ = เปิดบริบท (**ยังไม่มีการ์ดไหนถามข้อนี้**)
4. ✅ **OC-4530 ทับ OC-4363 หรือไม่ — ปิดแล้วด้วยหลักฐาน (พบ 2026-09-13 ตอนอ่าน OC-4363)**

## 🔑 OQ1 ปิดด้วยตัวหนังสือของ OC-4363 เอง — ไม่ต้องให้ PO ตัดสินว่า "ทับไหม" อีก

**Out of Scope ของ OC-4363 เขียนไว้เองว่า:** *"QR scan เพื่อ prefill code อัตโนมัติ (v1 = กรอกมือเท่านั้น)"*
และการ์ดนั้น**อยู่สถานะ In Progress แล้ว** (ไม่ใช่ To Do)

⇒ **ไม่ทับ แต่เป็นใบต่อ** · OC-4363 ดันงานสแกนออกไปเป็น v2 ตั้งแต่แรก และ OC-4530 คือใบที่มารับ v2 นั้น ·
คำเตือน "ห้ามลงมือทำขนานกัน" คลี่ออกเป็น **"OC-4530 รอ OC-4363"** ไม่ใช่ "เลือกใบใดใบหนึ่ง" ·
dependency ต้องเปลี่ยนจาก `relates` → **`blocked-by` OC-4363 + `blocked-by` OC-4335**

**รูปใหม่ที่เสนอสำหรับ OC-4530** = งาน FE ล้วน 4 ชิ้น: ตัวอ่านกล้อง → เติมรหัสลงช่องของ OC-4363 ·
fallback เมื่อไม่มี/ปฏิเสธกล้อง · ปิด stream จริงตอนออก · **รับ deep link** (OC-4335 mint **code + URL**
ไม่ใช่ code เปล่า) · **ลบ** Business Rule 1/2/3/5 และ AC เรื่อง QR ปลอม/ใช้ซ้ำ/เน็ตหลุด/ข้าม company ทิ้ง
เพราะ OC-4363 เขียนครบแล้ว (AC-02..AC-07 + ตาราง error + i18n key + audit payload) · เหลือ Rule 4 กล้อง + Rule 6 token

พ่วง: **design v3 ปุ่ม "สแกน QR ของร้านแทน" ใน `ovQR` จะชี้ไปท่าที่ถูกตัดทิ้ง ต้องแก้ copy** ·
epic ไม่ตรงกัน (OC-4363 ใต้ OC-2743, OC-4530 ใต้ OC-4349)

## ✅ PO เคาะแล้ว 2026-09-13 — ทางเลือก C แบบ B-lite

**OQ1 ของ OC-4539 ปิดแล้ว** · B = เพิ่มความสามารถใน backoffice เดิม **ไม่สร้าง role ใหม่ ไม่สร้างเว็บแยก**
(ต้นทุนของ B อยู่ที่ role ไม่ใช่หน้าจอ) · OQ4 "หน้าจออยู่ที่ไหน" ปิดตาม = backoffice เดิม

🔑 **PO เลือกท่าเป็น "ค้นหา" ไม่ใช่ "สแกน"** — ไอคอนแว่นขยายบน `TopNavBar.vue` → typeahead → การ์ดสมาชิก →
`/member/:member_code` · ต่างจากโจทย์ตั้งต้นที่เขียนว่า "ไม่ต้องค้นหาเบอร์" และ**ย้ายงานไปอยู่ฝั่งที่ติด consent**
(ทางสแกนไม่ติด consent เพราะลูกค้ายื่นบัตรเอง) → แบ่ง OC-4542 เป็น 2 เฟสเพื่อไม่ให้ค้างรอ OC-4361

### การ์ดหลังเคาะ

| การ์ด | หน้าที่ |
| --- | --- |
| **OC-4542** (ใหม่) | ค้นหาสมาชิกจาก top bar · เฟส 1 ชื่อ/รหัส/LINE name (ไม่ติด consent) · เฟส 2 เบอร์/อีเมล blocked-by OC-4361 |
| **OC-4539** (เขียนใหม่) | บันทึกมติ + เส้นสแกน: proxy verify ฝั่ง backoffice · ปุ่มสแกน · เอกสาร partner API · Size S–M แล้ว |
| **OC-4530** (เขียนใหม่) | v2 ของ OC-4363 เท่านั้น · blocked-by OC-4363 + OC-4335 |

### 🔴 ของจริงในโค้ดที่เพิ่งเจอ

`keyword_type` ของ `GET /company/:id/member` มีแค่ **email / line_display_name / phone**
(`backoffice-api/src/interface/fiber_server/spec/v1/spec.gen.go:632-635`) — **ไม่มี `member_code`**
ทั้งที่ route ปลายทางคือ `/member/:member_code` ⇒ พนักงานอ่านรหัสจากบัตรลูกค้าแล้วค้นไม่ได้ ต้องเพิ่มในสเปกแล้ว regen ·
และ endpoint นี้คืนโปรไฟล์เต็มทั้งแถว (ออกแบบมาเพื่อหน้าตาราง) จึงเอามาแปะ dropdown ตรง ๆ ไม่ได้

**typeahead ขยายช่อง enumeration** — ยิงทุกคีย์ · กติกาที่ลงใน AC: prefix ได้เฉพาะชื่อ+รหัส ·
**เบอร์/อีเมลต้อง match เต็ม** · ขั้นต่ำ 4 ตัว · debounce ≥300ms · rate limit · ห้าม log เบอร์เต็ม


## 🛠 Implement แล้ว 2026-09-13 (3 branch จาก origin/develop, ทำใน .worktrees/ ไม่แตะ checkout หลัก)

| Repo | Branch | SHA |
| --- | --- | --- |
| 3rdparty-api | `feat/oc-4539-internal-card-verify` | d9237e7 (merge codex/oc-4344-web-member-identity) · 5e3a714 |
| backoffice-api | `feat/oc-4542-oc-4539-member-search-card-verify` | c1644cb · 7f35af2 (review fixes) |
| frontend-backoffice | `feat/oc-4542-member-quick-search` | 6017ae2 |

**ลำดับ merge บังคับ: 3rdparty → backoffice-api → FE** (proxy verify ชี้ `/internal/v1/company/{id}/member/card/verify`
ที่มีแค่บน branch 3rdparty นี้) · **submodule ref ในโมโนรีโปยังไม่เลื่อน** — รอ PO
· 🔴 card-verify บน 3rdparty อยู่คนละ branch กับโครง `/internal/v1` (develop) จึงต้อง merge codex branch เข้า develop-base ก่อน

**สัญญา:** search `GET /v1/company/{id}/member/search?q=` → `{results:[{member_code, display_name,
name_source: profile|line|code, tier_name|null, point_balance}], total}` · 400 QUERY_TOO_SHORT / 429 RATE_LIMITED ·
phone/email-like → `[]` ไม่แตะ DB (เฟส 2 รอ OC-4361) · verify proxy 404 CARD_TOKEN_INVALID ทุกกรณี
· 3rdparty internal ตอบ 404 แต่ `/v2/openapi` เดิมตอบ **400** กับ sentinel เดียวกัน (ยังไม่เกลา)

**ยังไม่ทำ / follow-up:** เอกสาร partner API (OC-4539 ชิ้น 3) · SearchByPrefix ยัง select คอลัมน์ PII ที่ไม่ใช้ ·
HTTP client member_card ซ้ำโครง point_adjust · rate limiter in-memory ต่อ pod · FE type nameSource ยังไม่มี 'code' ·
**QA เบราว์เซอร์จริงยังไม่ได้ทำ** — SSO `accounts.dev-th` ปฏิเสธ origin นอก whitelist (localhost:5199) และ backoffice FE ไม่มี auth bypass
(มีแค่ VITE_BOLA_MOCK / VITE_THEME_MOCK) → ต้อง QA บน dev หลัง merge หรือรันบนพอร์ตที่ whitelist

⚠️ บทเรียนวันนี้: agent ตัว `developer` เผลอ spawn agent ซ้อนแล้วหยุดเอง ปล่อยให้ลูกเขียน worktree ต่อ — ต้องเช็ค `git status`+mtime ใน worktree
ก่อนเชื่อ notification "completed" · และช่วง classifier ล่ม agent review 5/6 ตัว stall ที่ 600s → รีวิวเองเร็วกว่า

## ลำดับงาน

ลำดับงานที่เสนอ: proxy verify (เล็ก) → ปุ่มสแกนบนหน้ารายชื่อสมาชิก (เล็ก, จบข้อ 1) →
รายการสิทธิ์ที่ใช้ได้ (กลาง, จบข้อ 3) → ค่อยเขียน OC-4530 ใหม่ → QR ท้ายใบเสร็จ (ใหญ่, แยกโครงการ)

**สถานะการ์ด ณ สิ้นวัน 2026-09-13:** OC-4530 + OC-4539 เขียน description ใหม่แล้ว · OC-4542 เปิดใหม่ ·
link: OC-4530 blocked-by OC-4363/OC-4335 · OC-4542 blocked-by OC-4361, relates OC-4539 ·
prototype มี 2 ชุด (ทิศ QR + จอค้นหา backoffice ที่พิมพ์ได้จริง)

⚠️ **OC-4363 สถานะเปลี่ยนเป็น `Blocked`** ระหว่างวัน (เดิม In Progress) — OC-4530 แขวนอยู่กับมัน

เชื่อม [[project_oc4529_member_card_token]] [[project_loyalty_point_cluster]] [[project_loyalty_canonical_contract]]
