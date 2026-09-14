---
name: project_oc4526_coupon_wallet
description: "OC-4526 กระเป๋าคูปอง — merge ครบ 4 ชั้นเข้า develop 2026-09-13; คูปองเป็นโดเมนของ engine, ตัดคูปองด้วยพนักงานสแกน, expired คำนวณตอนอ่าน; ยัง demo ไม่ได้เพราะไม่มีใครออกคูปอง"
metadata: 
  node_type: memory
  type: project
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-13T15:25:05.836Z
---

PO สั่ง "ทำทั้งหมด" 2026-09-13 → ผมเคาะ Open Question ทั้ง 4 ข้อของการ์ดเองแล้ว build ครบ 4 ชั้น
(ทุกข้อกลับได้ เขียนเหตุผลไว้ใน comment OC-4526 แล้ว)

## การตัดสินใจที่ถือเป็น contract ของฟีเจอร์นี้

1. **"ใช้เลย" = พนักงานสแกน** ไม่ใช่สมาชิกกดเอง — แตะแล้วเปิดหน้าที่มี QR + รหัส
   คูปองถูกตัดตอน staff ยิง `POST /v2/openapi/member/coupon/verify` เท่านั้น
   (สมาชิกกดเอง = กดพลาดครั้งเดียวเสียของที่คืนไม่ได้) · pattern เดียวกับบัตรสมาชิก [[project_oc4529_member_card_token]]
2. **DB เก็บแค่ `active | used | cancelled`** — `expired` **คำนวณตอนอ่าน** จาก `expires_at`
   ไม่มี cron/worker และไม่มีทางมีแถว active ที่จริง ๆ หมดอายุค้างอยู่ · จุดเดียวที่ derive คือ
   `model.Coupon.DisplayStatus(now)` ใน 3rdparty-api
3. **คูปอง = โดเมนของ engine (3rdparty-api)** เหมือน point/campaign/code/order · member-api เป็น BFF proxy อ่านอย่างเดียว
4. ทางเข้า = 3 จุดบน HOME ตาม design v3 · สวิตช์ `react/pages/Coupon/couponWallet.ts` → `COUPON_WALLET_ENABLED`

## contract (ตรวจเทียบทีละ field ก่อน merge — 3 agent สร้างคู่ขนาน)

engine `GET /me/coupons` → `{results:[…], total}` field `coupon_id`/`code`
→ BFF `GET /v1/me/coupons` → `{items:[…], total}` เปลี่ยนชื่อ 2 ฟิลด์เป็น `id`/`coupon_code`
→ FE parse `items`/`total` · `POST /v2/openapi/member/coupon/{verify,issue}` scope `member.coupon.verify` / `.issue`

## 🔴 ยัง demo ไม่ได้ (ไม่ใช่บั๊ก)

- **ไม่มีใครออกคูปอง** — `…/coupon/issue` เป็น seam ที่ไม่มี caller · เจ้าของคือ OC-4354 (แลกของรางวัล)
  กับการ์ดวงล้อลุ้นโชคที่ยังไม่มี ตาม Out of Scope ของ OC-4526 เอง → กระเป๋าว่างเสมอ
- ~~migration `019_create_coupon` ยังไม่ apply บน dev~~ → **apply แล้ว 2026-09-13 15:29 UTC**
  (`development_oc2plus_crm` บน `postgresql-pg18-0`) + เขียน ledger `/20260913002000-create-coupon`
  + เอาไฟล์เข้า repo migration ที่เป็น source of truth แล้ว (**MR !87**, คู่กับ !86 ของ card-token/news)
  · **staging/prod ยังไม่ได้รัน**
- **ฝั่งพนักงานไม่มี UI** — verify ต้องมี API key เรียก · OC-4530 ถูกเขียนขอบเขตใหม่เป็นกล้องฝั่งสมาชิก
  ไม่ใช่เครื่องมือที่เคาน์เตอร์อีกแล้ว

## ช่องว่างสองใบนี้มีการ์ดแล้ว (สร้าง 2026-09-14)

ยืนยันด้วย `git grep -ril coupon origin/develop` ว่า **backoffice-api และ backoffice FE มีโค้ดคูปอง 0 ไฟล์**
- **OC-4546** จัดการคูปองบน backoffice (ออก/ดู/ยกเลิก) — parent OC-2743 · เป็นใบที่ทำให้ OC-4526 demo ได้ครั้งแรก
  เพราะ `source_type=manual` ถูกกันไว้ให้แอดมินออกเองตั้งแต่ migration
- **OC-4547** พนักงานสแกนตัดคูปอง — parent OC-4349 · ต่อยอด `MemberCardScanner.vue` ของ OC-4539
- OC-4546 **blocks** OC-4547 (permission constant ร่วมกัน + ต้องมีคูปองก่อนถึงเทสได้)
- ร่างเต็ม: `docs/oc2plus/coupon-backoffice-cards-draft.md` · contract 3 ชั้น: `docs/oc2plus/coupon-backoffice-contract.md`
- **implement + เปิด MR แล้ว 2026-09-14** branch `feat/oc-4546-4547-coupon-backoffice` ทั้ง 3 repo
  3rdparty-api **!246** (internal surface) → backoffice-api **!579** (admin + Keto + audit) → FE **!607**
  · merge ตามลำดับนี้เท่านั้น ปลายน้ำเรียกต้นน้ำ · ทำใน git worktree `.worktrees/coupon` เพราะ shared checkout
  ติด branch ของ Codex อยู่ · ยังไม่เลื่อน submodule ref (convention ของ repo นี้คือเลื่อนหลัง merge เข้า develop)
- seed dev: `scripts/seed-coupons-dev.sh` (idempotent + `--reset`) — dev มีคูปอง 10 ใบครบ 4 สถานะ
  ผูกกับ member `B802-F171-0000-0001` ซึ่งเป็น **member คนเดียวทั้ง CRM บน dev** · `CPN-XCOM-0001`
  ผูก company ปลอมไว้เทสว่า company filter ไม่หลุด
- ✅ **PO เคาะ 2026-09-14: แยกเป็นสอง permission** — `oc2plus.coupon.manage` (แอดมิน: ออก/ดู/ยกเลิก, OC-4546)
  กับ `oc2plus.coupon.redeem` (พนักงานหน้าร้าน: ตัดคูปองอย่างเดียว, OC-4547) · `.redeem` ต้องไม่ implied จาก `.manage`
  · OC-4546 เป็นคนเพิ่ม constant **ทั้งสองตัวในรอบเดียว** เพราะขั้นที่แพงคือ publish repo ภายนอก
  `entity/access_control` ไม่ใช่จำนวน constant (+ ต้อง grant เข้า role เดิมด้วยมือ แบบ news.manage/OC-4356)
  · ผลข้างเคียงที่เขียนไว้ใน OC-4547 แล้ว: scanner ตัวเดียวคุมด้วยสองสิทธิ์ → คนที่มีแค่ member.view
  สแกนบัตรได้แต่สแกนคูปองไม่ได้ และต้องได้ข้อความ "ไม่มีสิทธิ์" ไม่ใช่ "คูปองใช้ไม่ได้"
- ⚠️ ยังต้องให้ PO เคาะ: ออกทีละคน vs bulk · แอดมินตั้งโค้ดเองได้ไหม · และ **ตาราง `coupon` ไม่มี field มูลค่า/ส่วนลดเลย**
- 🟡 OC-4543 เป็นการ์ดซ้ำของ OC-4526 (ovCoupons เหมือนกัน) ที่ถูกปิด Done ทั้งที่เนื้อการ์ดเขียนว่า backend ยังไม่มี

**How to apply:** ก่อนแตะคูปอง อ่าน 4 ข้อข้างบนก่อน — โดยเฉพาะข้อ 2 ห้ามเติม `expired` เป็นค่าที่เก็บใน DB
และข้อ 1 ห้ามเพิ่มปุ่มที่สมาชิกกดแล้วคูปองหาย

เชื่อม [[project_oc2plus_member_app_design_v2]] [[reference_oc2plus_member_frontend]] [[project_oc4530_counter_scan_reanalysis]]
