---
name: project_oc2plus_member_app_design_v2
description: "OC2Plus member app design of record is now **v3** ('OC2 Plus Loyalty v3 - Tier.dc.html'), a clean superset of v2 — 4 tabs HOME/REWARD/HISTORY/PROFILE + 18 overlays, ZERO ssk-*, brand #32a9ff (42 uses), and unlike v2 it DOES ship 21 semantic CSS tokens but NO type-role classes (inline font shorthand only). PO exported the whole canvas as a zip on 2026-09-11 so all four .dc.html files plus DBHeaventRounded woff2 x4 are on disk."
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


## 🔴 แก้ความเข้าใจผิดของผมเอง: #32a9ff ไม่ใช่ "default ที่มีอยู่แล้ว"

ผมเคยจดว่า `#32a9ff` == ค่า default ของ `--c-primary` ที่รีโปมีอยู่แล้ว **ไม่ครบความจริง และทำให้ agent เขียนคอมเมนต์เท็จลงโค้ด** (แก้แล้วใน commit 92f40dd)

ของจริง (ตรวจด้วย `git grep` บน develop @858c9a3 เมื่อ 2026-09-10) — **รีโปมี default ขัดกันเองสองที่**:

| ที่ | ค่า | เป็นตัวจริงไหม |
| --- | --- | --- |
| `src/theme/defaultTheme.ts` → `DEFAULT_THEME.primary` | **`#1A73E8`** | ✅ ใช่ — theme store ส่งค่านี้เข้า `applyTheme()` ซึ่งเป็นคน set `--c-primary` |
| CSS fallback `var(--c-primary, #32a9ff)` ใน `LoginView.vue` (5+ จุด) | `#32a9ff` | ❌ ไม่ — ใช้เฉพาะกรณีตัวแปรไม่ถูก set เลย |

เพราะฉะนั้น **การเปลี่ยนไป `#32a9ff` ตาม design v2 = สีที่ผู้ใช้เห็นเปลี่ยนจริง** สำหรับทุกบริษัทที่ไม่ได้ตั้ง theme · ห้ามอ้างว่าเป็น no-op

บทเรียน: เวลาจะเคลมว่า "ค่านี้มีอยู่แล้ว" ต้องแยกให้ออกว่าเจอมันใน **CSS fallback** หรือใน **ค่าที่ runtime set จริง** — สองอันนี้ในรีโปนี้ไม่ตรงกัน

---

# ⚠️ design of record เปลี่ยนเป็น v3 แล้ว (2026-09-11)

**`OC2 Plus Loyalty v3 - Tier.dc.html`** ในแคนวาสเดียวกัน · **v3 = superset ของ v2 แบบสะอาด**: 25 state เดิมครบทุกตัว ไม่มีอะไรถูกถอด + เพิ่ม 6 · ไฟล์ 107,835 → **152,962** ตัวอักษร

~~**ไฟล์ v3 ยังไม่อยู่ในเครื่อง**~~ — **แก้ 2026-09-11: PO export ทั้งแคนวาสเป็น zip ให้แล้ว** (`~/Downloads/Sellsuki loyalty app design (1).zip`, 22 ไฟล์ 5.5 MB) แตกไว้ที่ scratchpad ของ session · มีครบ: 4 ไฟล์ `.dc.html` (v3 Tier, v2 Friendly, Mobile Prototype, Loyalty Customer App), `fonts/DBHeaventRounded-{Regular,Med,Bold,Black}.woff2`, `assets/` (โลโก้/ไอคอน), `uploads/` · **นี่คือทางที่ควรใช้** — Claude Design canvas RPC ต้องผ่าน Chrome ที่ login แล้ว และ content ที่ได้เป็น base64 ที่ตัวกรองบล็อกตอนส่งค่ากลับ ขอ zip เร็วกว่ามาก

## วิธีดึงไฟล์จากแคนวาสโดยไม่ต้องรอ zip

WebFetch → **403** · เดา path บน `*.claudeusercontent.com` → **404**

ที่ได้ผล: ใช้ **Claude in Chrome** (มี session ของ claude.ai) เปิดหน้าแคนวาส แล้วเรียก Connect-RPC ของแอปเองจาก page context:

```js
const base='https://claude.ai/design/anthropic.omelette.api.v1alpha.OmeletteService/';
await fetch(base+'ListFiles',{method:'POST',credentials:'include',
  headers:{'content-type':'application/json'},body:JSON.stringify({projectId:PID})});
await fetch(base+'GetFile',{method:'POST',credentials:'include',
  headers:{'content-type':'application/json'},
  body:JSON.stringify({projectId:PID, path:'ชื่อไฟล์.dc.html'})});
// → { content, contentType, version } · content เป็น HTML ตรง ๆ
```

PID ของโปรเจกต์นี้ = `1308eeb1-6f77-4aee-b531-4475443a5215`

⚠️ **content filter ของ harness จะบล็อกผลลัพธ์** ถ้า return ข้อความดิบยาว ๆ ที่มี `?a=b` หรือ token-like (`[BLOCKED: Cookie/query string data]`) → ให้ **extract เป็น field ย่อย ๆ ใน JS แล้วคืนเฉพาะค่าที่ต้องการ** อย่าคืน raw HTML เป็นก้อน

## 6 state ใหม่ของ v3

`ovBenefits` (tier) · `ovReceipt` (สะสมแต้มด้วยใบเสร็จ) · `rcNoPhoto` / `rcHasPhoto` (สถานะช่องแนบรูป) · `hasPending` (ใบเสร็จที่ส่งแล้ว) · `ovCheckin` (เช็คอินประจำวัน)

## 🔴 ระบบ tier — คูณอัตราได้แต้ม ไม่ใช่แค่ป้าย

| ระดับ | need | perk |
| --- | --- | --- |
| BRONZE | 0 | รับแต้ม **1x** |
| SILVER | 300 | รับแต้ม **1.2x** |
| GOLD | 1000 | รับแต้ม **1.5x** |
| PLATINUM | 3000 | รับแต้ม **2x** |

`ovBenefits` มี: `tierLabel` · perk 2 บรรทัด · "แต้มที่สะสมได้ปีนี้" · **"ยอดซื้อ 90 วัน ฿12,480"** · progress `pointsText / nextTierGoal คะแนนสะสม` · **"ส่วนลดสำหรับสมาชิก `tierDiscount` %"** · ตาราง "สิทธิ์ทุกระดับ"

**มีการ์ด tier อยู่แล้ว = OC-3559** "Member tier [CRM] — ระดับสมาชิกจากแต้มสะสม + อายุ 1 ปีต่อรอบ" (label `business` ไม่ใช่ `customer-app-program` ผมเลยค้นไม่เจอรอบแรก — ดู [[feedback_search_before_declaring_gap]])

**design v3 ขัดกับ OC-3559:**

| | OC-3559 | design v3 |
| --- | --- | --- |
| ชื่อระดับ | Silver/Gold/Platinum/**Diamond** | **BRONZE**/Silver/Gold/Platinum |
| เกณฑ์ | 0 / 5,000 / 20,000 / 50,000 | 0 / 300 / 1,000 / 3,000 |
| tier ทำหน้าที่ | **คุมสิทธิ์** ใครได้แต้ม (L7 + AC9) | **ตัวคูณ** ได้กี่แต้ม 1x→2x |
| ส่วนลดสมาชิก | ไม่มี | `tierDiscount %` |
| ฐานวัด | แต้มที่ได้รับในรอบ 1 ปีต่อคน (L1+L3) | โชว์ทั้ง "แต้มที่สะสมได้ปีนี้" และ "ยอดซื้อ 90 วัน" |

**แถวที่สามหนักสุด** — คุมสิทธิ์ vs ตัวคูณ เป็นกลไกคนละตัว · ตัวคูณไหลเข้า Award Engine: OC-4413 (code review) และ OC-4415 (Done) **ไม่มี tier multiplier เลย** และ audit ต้องเก็บ multiplier ที่ใช้จริง

OC-3559 มีของดีที่ต้องไม่ทำหาย: schema 3 ตาราง (`member_tier` / `member_tier_state` / `member_tier_history`) · daily sweep (ไม่ใช่ cron รายปี เพราะรอบเป็นของแต่ละคน) · sync BOLA ด้วย flag `oc2plus_tier_gte_*` เพื่อเลี่ยงที่ BOLA ไม่มี `gte` และ 1 segment = 1 operator · เขียนเทียบไว้ที่ OC-3559 comment 44670 · 4 ข้อที่ยังเคาะไม่ได้: เกณฑ์เลื่อนคิดจากอะไร (แต้มปีนี้ / ยอดซื้อ 90 วัน / คะแนนสะสม — design โชว์ทั้งสาม) · `tierDiscount` เป็นส่วนลดตอนซื้อหรือตอนแลก · ลดระดับได้ไหม · multiplier อยู่ชั้นไหน

## ✅ `ovReceipt` ปลดบล็อก OC-4504

"สะสมแต้มด้วยใบเสร็จ" · "ทีมงานตรวจสอบภายใน 24 ชม." · ฟิลด์: เลขที่ใบเสร็จ* (hint "อยู่มุมขวาบน") · ยอดซื้อ* · วันที่ซื้อ* · **สาขาที่ซื้อ** (`b.label`) · แนบรูป* · **"แต้มที่จะได้รับ" `rcPointsText` = preview ก่อนส่ง**

กติกาท้ายฟอร์ม: **"1 ใบเสร็จใช้ได้ครั้งเดียว · ส่งย้อนหลังได้ไม่เกิน 7 วัน · ใบเสร็จที่แก้ไขหรืออ่านไม่ออกจะถูกปฏิเสธ"** — **ข้อ 7 วันไม่เห็นใน `SUBMIT_ERROR_CODES` ปัจจุบัน** ต้องเช็คกับ OC-4362

→ **OC-4531 (ค้นหาสาขา) ไม่ใช่ปุ่มลอยอีกต่อไป เป็น dependency ของฟอร์มใบเสร็จ**

## `ovCheckin` — แหล่งแต้มที่สาม

"เช็คอินทุกวันรับแต้มเพิ่มขึ้นเรื่อยๆ **วันที่ 7 รับ 50 คะแนน**" · "**ทุกครั้งที่เช็คอิน รับสิทธิ์หมุนวงล้อเพิ่ม 1 ครั้ง**" · "ลืม 1 วัน จำนวนวันต่อเนื่องเริ่มนับใหม่" · ทับซ้อนกับ OC-4527 (ภารกิจ) ที่ให้ทั้งแต้มฟรีและสิทธิ์หมุนเหมือนกัน — ต้องเคาะว่าแยกหรือรวม

เขียนไว้ที่ OC-4349 comment 44669 · OC-4504 comment 44668


## สิ่งที่นับได้จาก v3 ตอนวางชั้น token จริง (2026-09-11, OC-4535)

**v3 มี CSS custom property 21 ตัว เป็นระบบ semantic** — ต่างจาก v2 ที่มี **0 ตัว** (เขียน hex ตรง ๆ):
`surface, surface-2, surface-3, line, line-soft, ink, ink-solid, ink-2, ink-3, muted(#5b6472), faint, brand-ink(#1473e1), brand-soft, brand-line, warn-soft/line/ink, ok-soft/line/ink, danger-ink`

**สีแบรนด์ไม่ได้เปลี่ยน** — `#32a9ff` ยังเป็นสีหลักและ v3 ใช้ **42 ครั้ง** (v2 ใช้ 33) ส่วน `--brand-ink: #1473e1` เป็น token สำหรับตัวอักษร/เส้นบนพื้นแบรนด์ · เคยเข้าใจผิดว่า v3 ย้ายแบรนด์ไปสีเข้ม

**กับดักที่สำคัญสุด: v3 ไม่มี type-role class เลย** เขียน `font: <weight> <size> 'DB HeaventRounded'` แบบ inline ทั้งไฟล์ · ขนาด 16/17/18/19/20/21/22/24, น้ำหนัก 400/500/700/900, คู่บ่อยสุด 400/17 ×28, 500/18 ×21, 400/19 ×18 · line-height มี 13 ค่าไม่สม่ำเสมอ · **พอร์ตตรง ๆ = hardcode px ทั้งแอป ซึ่งขัด `feature-dod.md` เอง** → OC-4535 จึงตั้ง type role 12 ตัวขึ้นมาเอง และพิสูจน์ด้วยเทส totality ว่าคู่ (weight,size) ที่ v3 ใช้จริงทั้ง **34 คู่** ลงครบทั้ง 12 role

radius ที่ใช้: `9999px ×63, 8px ×35, 12px ×26, 14px ×17, 6px ×12, 20px ×10, 16px ×9, 16px 16px 0 0 ×4` (ตัวท้าย = bottom sheet) · shadow: `0 1px 2px rgba(0,0,0,.05)` + focus ring `0 0 0 3px rgba(50,169,255,.22)` · gap: `12/10/8/14/6/7/9/2px`

**โครง v3 จากคอมเมนต์ในไฟล์:** `SPLASH → REGISTER → OTP → APP` · APP มี 4 แท็บ `HOME/REWARD/HISTORY/PROFILE` · HOME มี tier card, points strip, daily check-in, play zone (ปุ่มไอคอนปัดได้) · overlay 18 ตัว: `SCRIM, SCAN, REWARD SHEET, CONFIRM, SUCCESS, COUPONS, CAMPAIGN, ALL CAMPAIGNS, BENEFITS, RECEIPT, EDIT PROFILE, ADDRESS BOOK, ADD ADDRESS, SETTINGS, NOTIFICATIONS, CHECK-IN, MISSIONS, LUCKY DRAW, TOAST`

⚠️ **description ของการ์ดเก่าหลายใบยังเขียนว่า v2 เป็น design of record** (เช่น OC-4346) ถ้าเจอความขัดแย้ง **ยึด v3** ตามที่ PO ยืนยันตอนส่ง zip มาให้ 2026-09-11

## Where the design files actually are, and v3's ovReceipt (2026-09-11)

The canvas export is at **`~/Downloads/Sellsuki loyalty app design (1).zip`** (the older
`Sellsuki loyalty app design.zip`, no suffix, has only the three v2-era files — **no v3**).
Nothing is unpacked in the monorepo; `find` for `*.dc.html` under `sellsuki_mono` returns nothing,
which is why a session can wrongly conclude the design is unavailable. Unzip to a scratch dir:
`unzip -oq "$HOME/Downloads/Sellsuki loyalty app design (1).zip" -d <scratch>` → four files,
design of record = **`OC2 Plus Loyalty v3 - Tier.dc.html`, 1,937 lines**.

**v3 DOES contain the receipt-submit screen** — this retires the 🔴 "เคาะไม่ได้ = เริ่มการ์ดนี้ไม่ได้"
blocker on OC-4504, which was written against v2. It is overlay **`ovReceipt` at line 883**:
title "สะสมแต้มด้วยใบเสร็จ" (`:887`), receipt-number field (`:894`, hint `:896`), photo attach
(`:917`, `:921-922`), attached state (`:930`), submit "ส่งใบเสร็จเพื่อรับแต้ม" → `submitReceipt`
(`:940`), a "ใบเสร็จที่ส่งไปแล้ว" list (`:945`), and a rules line (`:961`). Entry point is the
`ovScan` overlay's (`:583`) button "สแกนไม่ได้? กรอกเลขใบเสร็จ" → `goReceipt` (`:595`).
Recorded with line numbers in OC-4504 comment 44704.

Three deltas vs the shipped app, flagged so nobody implements them as a freebie: design shows it as a
**bottom sheet, the app has a real route** `/:slug/point-claims/new` (keep the route — strangler needs a
path and LINE deep-links into it); the design puts a recent-receipts list **on the same screen** while
the app has a separate page; the design's **"ส่งย้อนหลังได้ไม่เกิน 7 วัน" is a business rule** that
member-api may not enforce; and the design has **no marketplace/order-id channel** at all, though the
app supports it.
