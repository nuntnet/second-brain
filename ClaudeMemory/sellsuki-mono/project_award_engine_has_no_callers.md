---
name: project_award_engine_has_no_callers
description: OC-4413/4420 award engine สร้างเสร็จ เทสต์ครบ แต่ไม่มีใครเรียกเลยสักที่ — adapter ทุกใบ To Do และไม่มีการ์ดสำหรับต่อ point-claim approve เข้า engine
metadata: 
  node_type: memory
  type: project
  originSessionId: 1da93659-b2fa-4cc1-9205-a04c4be3ee2c
  modified: 2026-09-19T17:07:52.522Z
---

**ตรวจโค้ดจริง 2026-09-18: ไม่มีใครเรียก `award.Evaluate` และไม่มีใครเรียก
`CommitAward` เลยสักที่** เครื่องคิดแต้มอยู่ที่ 3rdparty-api
`src/use_case/award/` (evaluate.go + types.go — pure, `types.go` import แค่ `time`)
มี unit test ครบ แต่เป็นไลบรารีที่ไม่มีผู้เรียก

**สถานะ adapter (Jira 2026-09-18):** OC-4412 sync POS API = **To Do** ·
OC-4414 Kafka consumer = **To Do** (blocked-by OC-4286+OC-4287 ซึ่ง To Do ทั้งคู่
และตัวนั้น blocked-by PAT-2300 อีกชั้น) · OC-4335 QR live mint = **To Do** ·
OC-4422 preload lot = **To Do** · ส่วน OC-4413/4420 = "Ready to test (DEV)"

**Why:** การ์ดถูกแตกเป็น core + adapters โดยตั้งใจ แล้ว core เสร็จก่อน — ลำดับถูก
แต่ core ขึ้น "Ready to test (DEV)" ได้ทั้งที่ไม่มีคนเรียก เพราะ library card ไม่ถูก
บังคับด้วยกฎ `feature-dod.md` *"A backend with no caller is invisible"*

**How to apply:**
- **ช่องทางที่ส่งจริงแล้วทั้งสองใบไม่ได้ใช้ engine** — OC-4362 (ใบเสร็จ) และ
  OC-4407 (marketplace) ผ่าน backoffice-api `ApprovePointClaim` ซึ่งจ่าย
  `computeBaseRatePoints(amount, primary.BaseRate)` = **อัตราพื้นฐานล้วน หน่วยแต้ม
  หลักหน่วยเดียว ไม่มีแคมเปญเข้ามาเกี่ยวเลย** (คอมเมนต์หัวฟังก์ชันบอกเองว่า full
  engine เป็น follow-up) — **แต่ไม่มีการ์ด follow-up ใบนั้นอยู่จริง** (ค้นด้วย
  "award engine"/"Evaluate"/"Commit"/"อนุมัติ"/"คำขอแต้ม" ตั้งแต่ ส.ค. + เช็คคีย์
  4412/4413/4414/4420/4421/4422/4335/4407/4419 ตรง ๆ) ⇒ ต้องเปิดใหม่
- **adapter ตัวแรกที่ควรทำคือ point-claim approve ไม่ใช่ OC-4412** — ช่องทางเดียวที่
  ของพร้อมครบ (มีหน้าจอ มีคนใช้ มี line items จาก OCR) และไม่ต้องรอ integration
  ภายนอก ต่างจาก 4412/4414 ที่รอทั้งคู่
- งานต่อสายข้ามรีโป: engine อยู่ 3rdparty-api, approve อยู่ backoffice-api →
  เสนอให้ **backoffice-api เป็นเจ้าของการแจกแต้มตอนอนุมัติเอง** (ตาม
  [[oc2plus-service-boundary]] rule: actor คือสตาฟ) โดยเลื่อนแพ็กเกจ `award` ขึ้นไป
  อยู่ shared entity module — **ไม่** ให้ backoffice ยิง HTTP ไปหา 3rdparty
  ⚠️ ต้นทุน: สองรีโปอยู่คนละเวอร์ชันของ entity module (`v0.2.2` vs `v0.34.0`)
- `SYSTEM_BASE_EARN_{unit}` **ยังไม่มีใคร seed** — เจอแค่ในคอมเมนต์ (`IsSystem`
  flag ใน types.go + คอมเมนต์ใน point_claim_approve.go) ทางที่ง่ายกว่าคือปล่อยให้
  base rate เป็น field ที่อ่านตรง ๆ แล้วใช้ branch `IsSystem` จัดการ suppression
- **ไม่มี base rate config แล้วเกิดอะไร:** `campaign_rate` + `flat_per_receipt`
  ทำงานปกติ · `multiplier` ถูก skip ด้วย `SkipBaseRateDisabled` · ไม่มีแคมเปญด้วย =
  0 แต้ม · **รูที่เจอ: หน้า `/readiness` แถว `point_unit` เช็คแค่
  `HasActivePointUnit` ไม่ได้ดู base_rate_enabled** ⇒ บริษัทที่แจกแต้มไม่ได้เลยขึ้น
  เขียว (backoffice-api `src/use_case/company_readiness.go`)
- `ListResolvedSKUsActiveAt` (backoffice-api) รวม SKU ของทุกแคมเปญมากองเดียวแล้ว
  **de-dup แบบ "แคมเปญแรกชนะ" ทิ้ง campaign_id ไปเลย** ⇒ หน้ารีวิวตอบไม่ได้ว่า
  บรรทัดไหนเข้าแคมเปญไหน · และแคมเปญ `product_scope=all` ไม่มีแถวในตารางนี้เลย
  จึงมองไม่เห็นด้วยซ้ำ

**การ์ดที่เปิดเพื่อปิดช่องนี้ทั้งหมด (สร้าง 2026-09-19, parent OC-2743, To Do ทุกใบ):**
OC-4571 หน่วยเงินสตางค์ (ทำก่อนทุกใบ) → OC-4572 ยก `award` ขึ้น shared module →
OC-4573 pick-best ต่อหน่วยแต้ม (ถือ assumption D-4) → OC-4575 approve เรียก engine
→ OC-4576 preview API → OC-4577 หน้ารีวิว · OC-4574 campaign_id บน resolved SKU
(blocks 4573+4578) · OC-4578 guardrail ตอนสร้างแคมเปญ · OC-4579 คอลัมน์ช่องทาง +
แถวปักหมุดอัตราพื้นฐาน · OC-4580 readiness ดู base_rate_enabled ·
**เข้า sprint แรกได้ทันทีโดยไม่มี dependency: 4571 / 4574 / 4579 / 4580**

🔴 **อย่า merge MR !224 ของ OC-4339 (clawback) ก่อน OC-4575** — มันหา award จาก
`award_dedup_registry` ที่ฝั่ง point-claim ยังไม่มีใครเขียน ⇒ ใบเสร็จ/marketplace
ที่ยกเลิกจะหักคืนไม่ได้แบบไม่มี error (คอมเมนต์แจ้งไว้บน OC-4339 แล้ว)

**decision ที่ PO เคาะ 2026-09-19/20 (อยู่ในการ์ด แต่ยังไม่มีในโค้ด — re-derive จากโค้ดไม่ได้):**
- **`award` ไปอยู่โมดูล `gitlab.sellsuki.com/sellsuki/oc2plus/line-crm/backend/entity`** ไม่ใช่โมดูลกลาง `sellsuki/sellsuki/backend/entity` — เพราะสองรีโปใช้โมดูล CRM ร่วมกันที่ **v1.9.7 เท่ากันอยู่แล้ว** (go.mod:28 ทั้งคู่) จึงไม่ต้อง bump ข้ามเวอร์ชัน ส่วนโมดูลกลางจะบังคับให้ 3rdparty กระโดด v0.2.2 → v0.34.0 = 32 minor · ลำดับ merge: entity → 3rdparty → backoffice
- **flag `POINT_CLAIM_CAMPAIGN_AWARD` คุม *ข้อมูลเข้า* ไม่ใช่สลับ *เส้นทาง*** — ปิด = เรียก Evaluate/Commit เหมือนเดิมแต่ป้อนเฉพาะชั้นอัตราพื้นฐาน (ได้ตัวเลขเท่าวันนี้เป๊ะ) · เปิด = ป้อนแคมเปญด้วย ⇒ เหลือเส้นทางเดียว ลบ `computeBaseRatePoints` ได้จริง, proved-red มีความหมาย, rollback ปลอดภัย
- **พรีวิวค้างแล้วแคมเปญหมดอายุ = 409 พร้อมตัวเลขใหม่ ให้กดยืนยันอีกครั้ง** ห้ามแจกตามตัวเลขใหม่เงียบ ๆ
- **readiness แถว `point_unit` เขียวได้สองทาง** — อัตราพื้นฐานเปิด **หรือ** มีแคมเปญได้แต้ม active (ร้านที่แจกผ่านแคมเปญล้วนเป็นการตั้งค่าที่ถูกต้อง) · Blocking เมื่อไม่มีทั้งสองทาง · 🔴 **ไม่มี per-company toggle ของฟีเจอร์คำขอแต้มอยู่จริง** — กวาด `Enabled` ทั้ง model layer สองรีโปแบบไม่กรองแล้วเจอตัวเดียวคือ `PointBaseRate.Enabled` (`point.go:42`) อย่าไปตามหาหรือประดิษฐ์ใหม่

ดู [[project_loyalty_overlap_best_single_campaign]] [[project_loyalty_point_cluster]]
[[project_oc4362_approve_and_admin_edit]] [[project_oc4551_readiness_checklist]]
