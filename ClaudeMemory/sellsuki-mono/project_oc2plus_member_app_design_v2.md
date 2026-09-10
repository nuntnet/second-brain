---
name: project_oc2plus_member_app_design_v2
description: "OC2Plus member app design of record = 'OC2 Plus Loyalty v2 - Friendly.dc.html' (4-tab app, 11 overlays); it uses ZERO ssk-* DS custom elements, brand #32a9ff == the app's existing --c-primary default; PO decided 2026-09-10 to port straight to this design in one pass"
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
isOtp → isApp` with a 6-box OTP and `otpError` · overlays `ovQR ovScan ovStamp ovReward ovCoupons
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
