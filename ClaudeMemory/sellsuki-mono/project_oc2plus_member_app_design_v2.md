---
name: project_oc2plus_member_app_design_v2
description: "OC2Plus member app design of record = 'OC2 Plus Loyalty v2 - Friendly.dc.html' (4-tab app, 10 overlays); it uses ZERO ssk-* DS custom elements, brand #32a9ff == the app's existing --c-primary default; PO decided 2026-09-10 to port straight to this design in one pass"
metadata:
  node_type: memory
  type: project
---

**Decided 2026-09-10 (PO).** Design bundle "Sellsuki loyalty app design" (Claude Design canvas,
`claude.ai/design/p/1308eeb1-6f77-4aee-b531-4475443a5215`) holds **three** `.dc.html` files:

| file | state machine | verdict |
|---|---|---|
| `Loyalty Customer App.dc.html` | none (no `sc-if`) | older/static |
| `OC2 Plus Loyalty - Mobile Prototype.dc.html` | 25 states | what the shared URL points at |
| **`OC2 Plus Loyalty v2 - Friendly.dc.html`** | 25 + `ovMissions` + `ovLucky` + "ฟรี" | **design of record** |

Zip mtimes are all identical (zip creation time) — they cannot tell you which is newest.

**Structure (v2):** 4 tabs `หน้าแรก / ของรางวัล / ประวัติ / โปรไฟล์` · auth `isSplash → isRegister →
isOtp → isApp` with a 6-box OTP and `otpError` · overlays `ovQR ovScan ovReward ovCoupons
ovCampaign ovCampaignsAll ovConfirm ovSuccess ovMissions ovLucky` · `selPhysical`/`isShip` (physical
reward → address + shipping) · `showLiffBar` · `historyEmpty` · `toast` · `hasScrim` · language TH/EN
in profile.

## The two findings that reshaped the cards

1. **`grep -c "ssk-"` = 0 in all three files.** The design uses utility CSS + self-hosted
   `DB HeaventRounded` woff2 (400/500/700/900) + Lucide icons — **no design-system custom elements**.
   This inverted OC-4493 from "upgrade `sellsuki-components` to the scoped package, don't change the
   visual" into "lay down design tokens and stop depending on `ssk-*`".
2. **Brand `#32a9ff` (33 uses) is exactly the app's existing `--c-primary` default**, with
   `#1473e1` as the dark shade. So per-company theming (`themeStore.loadTheme(slug)`) still layers
   over the design unchanged — no new theming mechanism needed. Fonts are the same family too.
   Design colors are raw hex with **no CSS custom properties**, so tokenising them is real work.
   Other tokens: neutrals `#e5e7eb #1f2937 #6b7280 #9ca3af #374151 #f3f4f6 #f9fafb #4b5563`,
   danger `#e11d48`, radii pill `9999px` + 8/12/6/14/16/20 + bottom-sheet `16px 16px 0 0`.

## Consequence for the strangler lane

**PO chose: port straight to the new design, one pass** (not port-then-restyle). So every
"หน้าตาเหมือนเดิม / เทียบ screenshot กับ Vue เดิม" AC in OC-4501..OC-4505 is invalid, and the
OC-4491 baseline becomes a **behaviour** contract: assert flow outcomes via `data-testid`/role,
never exact copy or color on a page slated for redesign (amended in OC-4491 comment 44650).

## Scope the design covers that OC-4344 does not

- Already carded but under **OC-4349 (Epic 2)**: OC-4352 catalog โปรโมชั่น+ของรางวัล (list+detail),
  OC-4354 แลกแต้ม — so the design spans two epics.
- **No card found** on the OC board (non-Done, searched summary ~ แสตมป์/stamp/ภารกิจ/mission/สาขา/
  QR/คูปอง/แลก): stamp card, ค้นหาสาขา (branch finder), member-app address+shipping for physical
  rewards, language switch in profile, and v2-only **ภารกิจ (missions)** + **lucky draw**.
- OC-4346's nav i18n keys (`shell.nav.points/promotions/news/profile`) do not match the design's
  four tabs — there is no news tab; ของรางวัล and ประวัติ take its place.

Related: [[project_oc2plus_member_react_migration]] [[project_oc2plus_liff_shell_is_the_line_entry]]

## Card sweep completed 2026-09-10 (summary on OC-4344, comment 44653)

Rewrote: OC-4370, 4346, 4490, 4491, 4492, 4493, 4494, 4496, 4499, 4501, 4502, 4503, 4504, 4505.
AC-amendment comments only (body was ~90% right): OC-4500 (44651), OC-4506 (44652).
Verified already-correct, untouched: OC-4495.

**Card claims that did not match the code** (all cards were written 2026-09-09 and drifted or were
invented; treat every file:line in this lane as needing re-verification before use):

- **OC-4505 cited seven `liff.*` call sites in `HomeView.vue` that do not exist.** The LIFF SDK is
  imported once (`src/stores/integration/index.ts:5`) and called from exactly two places:
  `liffInit` / `liffGetAccessToken` in that store, and `liff.logout()` in
  `src/views/Error/ErrorKindView.vue:69`. `HomeView.vue` only reads the `liff.state` query param
  (`:51-52`) and names a Sentry tag `'liff.sendMessages.welcome'` (`:150`).
- OC-4494: `services/auth` is **103** lines, not 71. And `stores/auth/index.ts:79` has
  `else phoneError.value = 'invalid'` — a fall-through that flattens `member_not_found`,
  `otp_expired`, `otp_locked`, `otp_invalid` into "wrong phone number". Preserve or change it
  deliberately; do not let a refactor silently alter it.
- OC-4496: every line reference had shifted, and it omitted `isForbidden` (`:54-56`). `fetchDetail`
  now splits **four** ways (401 `:161` / not_found `:162` / forbidden `:163` / network `:164`) —
  a usecase returning fewer states silently reverts OC-4498.
- LOC drift: PointClaimList 783→**795**, PointClaimDetail 798→**812**, HomeView 189→**197**.
  Error views are **four** files now (NotFound 34, ErrDataAccessDenied 38, ErrUnexpected 38,
  **ErrorKindView 115**) with a new route `/error/:kind` (`router:85`).
- Router has **10** routes, not 9 (OC-4500/OC-4506 both said 9).
- OC-4502: design adds a **birthday** field — `birthdate` exists in the DB and repository
  (`member_repository/postgresql.go:30`, column `i.birthdate`) but **not in `v1.yaml`**, so it needs
  an API change. Design also merges first/last name into one input while the API takes them apart.
- OC-4504: there is **no client-side file size/type check** — only `accept=...` at
  `PointClaimNewView.vue:768`; rejection comes back from the server as `photo_invalid`.
  `DRAFT_TTL_MS` is 24h (`pointClaimDraft.ts:22`), drafts live in **IndexedDB**, and an expired one
  is deleted by `loadDraft` (`:140`), not merely hidden. `SUBMIT_ERROR_CODES` (`:58-63`) holds five
  codes plus `unauthenticated` = six outcomes.

**Three cards are blocked on a design question, not on engineering:** design v2 has no login screen
(OC-4501), no claims-list counterpart (OC-4503 — `tabHistory` reads as points history, a different
domain), and no receipt-submit screen (OC-4504 — only `ovScan`). Each says "cannot decide = cannot
start" rather than guessing.


## Card sweep — OC-4352 / OC-4354 (2026-09-10)

**OC-4352** โปรโมชันกับของรางวัลเป็น **สอง surface แยกกัน** (`tabReward` vs `ovCampaignsAll`) ไม่ใช่หน้าเดียว · ตัวกรองหมวดที่การ์ดเดิมเขียนว่า "ไม่อยู่ใน MVP" อยู่ในขอบเขตแล้ว (design มี `cats` 5 + `campCats` 5 + toggle "แสดงเฉพาะที่แลกได้") · รายละเอียดของรางวัลเป็น **overlay ไม่ใช่ route** → กระทบ route table ของ OC-4500

**OC-4354** design มีขั้น `selPhysical` "รับสินค้าอย่างไร" → **รับที่หน้าร้าน / จัดส่งที่บ้าน** ซึ่ง **ยกเลิก Rule 4 เดิม** ที่บังคับ `shipping_address_id` กับ product ทุกกรณี — ตอนนี้บังคับเฉพาะ `delivery` · ต้องมี field ใหม่ `fulfilment_method` (`pickup`|`delivery`) ที่ contract ยังไม่มี → เคาะไม่ได้ = เริ่มไม่ได้ · `isShip` แสดง "แต้มคงเหลือหลังแลก" เป็น preview ก่อน confirm (ค่าจริงต้องมาจาก response) · `ovCoupons` "คูปองของฉัน" (รหัสแบบ `CPN-4F82-90KD` + วันหมดอายุ + "ใช้เลย") คือ artifact หลังแลกที่ยังไม่มีการ์ดและ contract ไม่ได้คืนมา

ช่องว่างทั้ง 4 ข้อของ contract เขียนไว้ที่ [[reference_oc2plus_jira_project]] OC-4347 comment 44654 (BFF เป็นเจ้าของ ไม่ใช่การ์ดหน้าจอ)

## รายชื่อ state ที่ถูกต้อง (ยืนยันด้วย grep `<sc-if value="{{ … }}"` เมื่อ 2026-09-10)

**25 ตัว** (ไม่ใช่ 27): `showLiffBar isSplash isRegister isOtp otpError isApp tabHome tabReward tabHistory historyEmpty tabProfile hasScrim ovQR ovScan ovReward selPhysical isShip ovConfirm ovSuccess ovCoupons ovCampaign ovCampaignsAll ovMissions ovLucky toast`

**🔴 `ovStamp` / แสตมป์ ไม่มีในไฟล์ v2 เลย** — grep ไม่เจอทั้ง `stamp` และ `แสตมป์` · ผมเคยจดผิดและเคยเขียนลงการ์ด OC-4352 ด้วย (แก้ไว้ใน OC-4349 comment 44659) · ถ้าเห็นแสตมป์ในบันทึกเก่า มันมาจากไฟล์อื่นในชุด ไม่ใช่ design of record

## เนื้อหาที่ดึงจากไฟล์จริง (ใช้อ้างได้ ไม่ต้อง grep ซ้ำ)

* `ovCoupons` — "คูปองของฉัน" + `c.name`/`c.exp`/`c.code` + ปุ่ม "ใช้เลย" (ไม่มีหน้าถัดไปหลังกด)
* `ovMissions` — "ภารกิจของคุณ / ทำภารกิจง่ายๆ ได้แต้มฟรี / **ไม่ต้องซื้อของก็สะสมแต้มได้** / ภารกิจรีเซ็ตใหม่ทุกวันเที่ยงคืน / วันนี้ทำไปแล้ว `missionDoneText`" + สองกลุ่ม "ภารกิจประจำวัน"/"ภารกิจพิเศษ" (`m.title`/`m.sub`/`m.btnLabel`)
* `ovLucky` — "หมุนฟรีวันละ 1 ครั้ง" + `wheelCenter`/`spinBtnLabel`/`spinNote` + "รางวัลในวงล้อ" (`p.label`/**`p.chance`** = โชว์โอกาสให้ผู้ใช้เห็น) + "สิทธิ์หมุนเพิ่มได้จากภารกิจประจำวัน · **รางวัลที่ได้จะเข้ากระเป๋าคูปองทันที**"
* `ovQR` — "บัตรสมาชิก" + `memberId` + `userName` · **`tier`** + "ให้พนักงานสแกนเพื่อสะสมหรือใช้แต้ม" + **"รหัสรีเฟรชทุก 60 วินาที"** + "สแกน QR ของร้านแทน"
* `ovScan` — "สแกนรับแต้ม / วางกล้องให้ตรงกับ QR ที่เคาน์เตอร์ / **หรือแจ้งเบอร์โทรกับพนักงานแทนได้**" (+ ปุ่ม "จำลองสแกนสำเร็จ" ที่เป็นของ prototype)
* `tabProfile` แถวต่าง ๆ — `pDob` (วันเกิด) · `pAddr` (ที่อยู่) · **ภาษา ไทย/English** · การแจ้งเตือน (`t.label`/`t.sub`)
* `tabHome` — ปุ่ม "ค้นหาสาขา" (`#1473e1` บน `#bce8ff`) **ที่ไม่มีหน้าปลายทางใน design**
* reward data มี `cat:'ลุ้นโชค'` และ "สิทธิ์ลุ้นโชคประจำเดือน" cost 50 → สิทธิ์หมุนซื้อด้วยแต้มได้

## การ์ดที่เปิดจากช่องว่างเหล่านี้ (2026-09-10)

OC-4526 คูปอง · OC-4527 ภารกิจ · OC-4528 วงล้อ · OC-4529 บัตรสมาชิก QR · OC-4530 สแกน · OC-4531 ค้นหาสาขา — ทั้งหมดใต้ epic OC-4349, ยังไม่ผ่าน DoR, ไม่มี sprint

**ยังไม่เปิดการ์ด (รอเช็ค/เคาะ):** สลับภาษา+การแจ้งเตือน (เช็ค OC-4360 ก่อน) · ระบบ tier (ไม่มีใน backlog เลย) · **เครื่องมือฝั่งพนักงานสแกน (ไม่มีการ์ด ไม่มีเจ้าของ — ถ้าไม่มี OC-4529/4530 ทำเสร็จก็ใช้ไม่ได้)**
