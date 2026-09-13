---
name: reference_oc2plus_member_app_local_qa_session
description: วิธีเปิด customer app (member frontend) ในเบราว์เซอร์แล้ว login เป็นสมาชิกจริงบนเครื่อง — env.development.local + test secret + OTP 000000 + slug localtest; 3 ใน 4 แท็บยังพังเพราะ consent/integration_id
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa22ebb1-f715-4667-85a1-61ebbcc816ab
  modified: 2026-09-13T04:16:17.320Z
---

ทำได้จริงเมื่อ 2026-09-12 ใช้เวลาหาพอสมควร — จดไว้ให้ไม่ต้องไล่ใหม่

## สูตรที่ใช้ได้

1. **`.env.development.local`** ใน `frontend/oc2plus-linecrm-frontend-member`
   (gitignore ครอบด้วย `*.local` อยู่แล้ว — ตรวจด้วย `git check-ignore -v`):
   ```
   VITE_SERVICE_MEMBER_BASE_URL=
   VITE_LOCAL_DEV_MEMBER_API_TARGET=http://localhost:8102
   VITE_LOCAL_DEV_TEST_SECRET=<ค่า TEST_KEY ของ member-api/.env>
   ```
   `vite.config.ts` อ่านตัวกลางเป็นสวิตช์เดียว แล้ว proxy `/v1` + `/system`
   ให้เป็น same-origin (จำเป็น เพราะ member-api ตั้ง CORS `*` ซึ่งเบราว์เซอร์
   ปฏิเสธเมื่อส่ง cookie) และ **ฉีดเฮดเดอร์ `X-Test-Secret` ให้อัตโนมัติ**

2. **login ผ่าน UI ได้เลย** ที่ `/<slug>/login` → เบอร์ → **OTP `000000`**
   `RequestMemberOTP` เห็น test mode แล้ว **ข้าม messaging service ทั้งก้อน**
   (`member_liff.go:96`) → ไม่มี SMS จริงออกไป · โค้ดคงที่อยู่ที่
   `member_liff.go:18 testModeOTPCode = "000000"`

3. **slug ที่ใช้ได้บนเครื่อง = `localtest`** → company `11111111-1111-4111-8111-111111111111`
   มาจากตาราง `oc2plus_bola_bindings` (ไม่ใช่ CCS) · `ResolveCompanyBySlug`
   **ไม่ได้กรองสถานะ** แถวนั้น `binding_status='failed'` ก็ยังใช้ได้
   สมาชิกที่มีเบอร์: `0899999999` (Local Tester, member_code M-LOCAL-0001),
   `0895556666`, `0818800346`

⚠️ เฮดเดอร์ test มี **สองชื่อ** ไม่เหมือนกัน: `members_v1.go` อ่าน `X-Test-Secret`
(ตัวที่ proxy ฉีด) ส่วน `auth_v1.go`/`member_v1.go` อ่าน `X-TESTING-SECRET`
(เส้น LINE access token) — สะกดผิดตัวแล้ว test mode เงียบ ๆ ไม่ทำงาน

## dev server: 5183 มักโดนจองอยู่แล้ว

เซสชันอื่นรัน `web-member` บน 5183 อยู่ — เพิ่ม entry `web-member-qa`
(port 5190, `--strictPort`) ไว้ใน `.claude/launch.json` **ที่ราก monorepo**
แล้ว · `preview_start` อ่านเฉพาะไฟล์ที่ราก ไม่อ่านของ submodule

## ⚠️ ตั้งแต่ 2026-09-13 (OC-4361, FE `36f0895`+`02f729d`): consent gate ครอบทุกหน้าที่ต้อง login

`WebConsentGate` ถูกย้ายจาก `/address` ไปนั่งที่ `ShellPageImpl` (ระหว่าง AuthGuard กับ ShellFrame)
→ session เว็บ/OTP ที่บริษัทยัง**ไม่มีแถวในตาราง `consent`** จะเจอหน้า
"บริษัทนี้ยังไม่ได้ตั้งค่าเอกสารสมาชิก กรุณาติดต่อฝ่ายสนับสนุน" **ตั้งแต่ HOME** ไม่ใช่แค่สมุดที่อยู่
(fail-closed ตาม OC-4340 — ตั้งใจ) · session LIFF ไม่ผ่าน gate นี้ (`useIsLineClient()`)

**ผลต่อสูตร QA ข้างบน:** login ด้วย `localtest` ได้ แต่จะ**ไปไม่ถึงแท็บไหนเลย**จนกว่าจะ seed
consent docs — ถ้าเห็นหน้านั้นอย่าไปไล่หา bug ใน FE

**สูตร seed (ทำแล้ว 2026-09-13, ได้ผลทันที):** consent ผูกกับ **integration** ไม่ใช่ company — session เว็บ/OTP
มี `integration_id=''` แล้ว member-api `GetWebConsentIntegrationID` (`integration_repository/postgresql.go:267`)
เลือก integration active ของบริษัทที่มี**ตัวเดียว** (`localtest` = `22222222-2222-4222-8222-222222222222`) ·
ตาราง `consent` (PK `integration_id, consent_type`) มีแถวของ integration `5d0e0b94-…` ("LOCAL TEST consent only",
บริษัท `60daa2a8-…`) ชี้เอกสารใน consent service ที่มีจริง → ยืมเลข consent_id เดียวกัน:
```sql
insert into consent(integration_id, consent_type, consent_id, created_at, updated_at)
values ('22222222-2222-4222-8222-222222222222','pdpa','434401',now(),now()),
       ('22222222-2222-4222-8222-222222222222','tos','434402',now(),now());
```
แล้ว reload → เกทแสดง PDPA/ข้อกำหนด (ตัวอย่าง) ให้กดยอมรับ 2 ฉบับ → เข้า HOME ได้ (`/me/point` ฯลฯ ตอบ 200 แล้วด้วย
เพราะ `""` ไม่ถึง column uuid อีก) · ถ้าบริษัทมี integration active >1 จะได้ `ErrPointClaimIntegration` แทน

ต้น 404 คือการอ่านเอกสาร `GET /v1/me/consent/pdpa` (status ตอบ 200) — และก่อน `02f729d`
`getDocument()` เป็นเมธอดเดียวใน `services/consent` ที่ไม่ห่อ `errorHandler` ทำให้ 404 กลายเป็น
`network` + ปุ่ม Retry ที่กดไม่มีวันผ่าน ทั้งที่เทส 29/29 เขียว (ไม่มี `describe('getDocument')`)
— ตัวอย่างสดของ [[reference_test_stub_more_permissive_than_service]]

consent gate **มีตัวเดียว = `WebConsentGate`** (PO เคาะ 2026-09-13) · ชุดเก่า
`ConsentGate/useConsentGate/ConsentModal` + chain ของมัน (`ConsentViewer`, `useConsentKeywords`,
`i18n/consentKeywords`, `consent.css`, `usecases/consent/versionRenewPort`) **ถูกลบไปแล้ว 12 ไฟล์**
— ถ้าเจอชื่อพวกนี้ในบันทึกเก่า/คอมเมนต์ อย่าไปหา · ที่ยังอยู่จริง: `WebConsentGate`,
`webConsent.css`, `ConsentRecovery` (Code + RewardRedemption ใช้), `ConsentDocuments` (Profile ใช้)

## สิ่งที่เห็นแล้วต้องไม่ตกใจ: 3 ใน 4 แท็บขึ้น error

`/v1/me/profile` = 200 แต่ `/v1/me/point`, `/me/point/expire`,
`/me/point/transaction`, `/me/campaigns` = **502** ทั้งหมด

ไม่ใช่บั๊ก FE และไม่ใช่ member-api ล่ม — member-api proxy ไป 3rdparty-api
(8103) ซึ่งตอบ **500** กลับมา สาเหตุที่ก้นบึ้งคือ
[[reference_oc2plus_otp_session_fails_3rdparty_consent]]: session ของ web/OTP
มี `integration_id = ""` แล้ว consent lookup เอาไปยิงเป็น uuid →
`pq: invalid input syntax for type uuid: ""` (`consent.go:48`)

**ต่างจากที่เคยจดไว้**: เดิมบันทึกว่าอาการคือ 403 `member_not_accept_consent`
ของจริงตอนนี้คือ **500 unexpected_error** เพราะ `""` ไปถึง column uuid ก่อน
จะได้ ConsentNotFound ด้วยซ้ำ — ต้นเหตุเดียวกัน แต่เสิร์ชด้วยคำว่า 403 จะไม่เจอ

ซ้ำร้าย **ตาราง `consent` ในเครื่องว่างเปล่า (0 แถว)** และ chain ยังต้องต่อไป
`consentRepository.GetConsentByID` = service `sellsuki-service-consent` อีกตัว
→ seed แถวเดียวไม่พอ **ปลดล็อกในเครื่องไม่คุ้ม** ถ้าอยากดูหน้า point/reward
จริง ๆ ต้องแก้ consent lookup ฝั่ง 3rdparty-api ซึ่งเป็น **การตัดสินใจเชิง
product** (สมาชิกที่ login ด้วยเบอร์โดยไม่มี LINE ถือว่ายินยอมหรือยัง) ยังไม่มีการ์ด

## เครื่องมือ

- DB: `docker exec -i sellsuki_mono-postgres-1 psql -U postgres -d oc2plus_crm`
  (docker อยู่ `/usr/local/bin/docker` เครื่องนี้ไม่มี `psql`)
- log ของ 3rdparty-api: overmind แยก socket ต่อเซสชัน — หา window ด้วย
  `tmux -L <sock> list-panes -a -F "#{window_name}"` แล้ว `capture-pane -p -t oc2plus-3rd -S -200`
  `oc2plus-api` = backoffice (8089) · `oc2plus-3rd` = 8103 · `oc2plus-member` = 8102

เชื่อม [[reference_oc2plus_member_frontend]] [[reference_oc2plus_member_api_test_mode_login]]
[[project_oc2plus_customer_bff_reads_direct_not_proxy]]
