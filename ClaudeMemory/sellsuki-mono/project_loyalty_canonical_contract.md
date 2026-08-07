---
name: project_loyalty_canonical_contract
description: "Loyalty cluster has a canonical contract sheet on OC-4413 — order_ref/channel/resolved_skus are defined there, not per-card"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-07T15:50:15.378Z
---

**OC-4413 (Award Engine Core) = hub + single source of truth ของ loyalty cluster.** Contract กลางประกาศไว้ที่คอมเมนต์ "CANONICAL CONTRACT SHEET" (2026-08-07) — การ์ดอื่นห้ามนิยามเอง ให้อ้างมาที่นี่

1. **dedup key = `(company_id, order_ref)`** — ชื่อ field ต้องเป็น `order_ref` ทุกใบ (เดิม 4412 ใช้ `order_id`, 4335 ใช้ `order_serial`, 4362 ใช้ `receipt_number`, 4407 ใช้ marketplace id → prefix ด้วย channel)
2. **`channel` enum เดียว:** `pos|online|line_oa|web|marketplace|manual|unknown` — config เพิ่ม `all` ได้ แต่ event ห้ามส่ง `all`
3. **product scope = SKU เท่านั้น** — BO (มี PIS client) expand category → `resolved_skus[]` snapshot ตอน publish; 3rdparty-api ไม่มี PIS client จึงห้ามรู้จัก category
4. **base rate = always-on ผ่าน implicit system campaign** `SYSTEM_BASE_EARN_{point_unit}` (เพราะ **ทุก award endpoint ผูก campaign เสมอ** — v1/v2 confirm ทุกเส้น require campaign_code) → engine ไม่ต้องมี branch "award ไม่มี campaign"; ตัวคูณ suppress base เฉพาะส่วน eligible
5. **decision ที่ยังค้างจริงเหลือ 2:** นิยาม "สมาชิกใหม่" · cumulative running total เมื่อ refund/clawback

**OC-4415 ย้ายไปเป็น field ในหน้าตั้งค่าหน่วยแต้มเดิม (เครื่องมือ > คะแนน) ไม่เปิดเมนูใหม่** — base rate = คุณสมบัติของ point currency (`point.Point` มี IsPrimary อยู่แล้ว, เพิ่ม 5 field: base_rate_enabled / base_amount_per_point / base_points_earned / base_rounding / base_rate_effective_from)

**การ์ดใหม่ 2026-08-07:** **PAT-2604** (บอร์ด PAT, epic PAT-2490) = สัญญา purchase event ขอ `line_items[{sku,qty,line_amount}]` + sku ตรง PIS + customer.phone E.164 — blocks OC-4414; ต้อง fold เข้า **PAT-2300** (Kafka event v2 = SoT ของ schema) · **OC-4419** = integration guide รวมชุด (decision tree + error playbook) เพราะ "swagger" เดิมเป็นแค่ DoD บรรทัดเดียวกระจายหลายใบ

**🔴 "หมวดสินค้า" ไม่มีอยู่จริงทั้งแพลตฟอร์ม (verified 2026-08-07):** `grep -i categor` = 0 ผลลัพธ์ ทั้ง pis-api (`src/entity`, `src/use_case/repository`), catalog-service, และ backoffice product layer · PIS `GetProductPaginateQueryParam` (`pis-api/src/interface/fiber_server/model/product_get_model.go:51`) = `Keyword/Type/RefID/Offset/Limit/SortBy/SortDirection/LocationID` ไม่มี category · backoffice client (`backoffice-api/src/repository/product_repository/rest.go:109-115`) ส่งได้แค่ `type`+`keyword` → **campaign "เลือกทั้งหมวด" ทำไม่ได้ ตัดออกจาก v1** (ต้องเปิดการ์ดฝั่ง PIS ก่อน) · พ่วง: **`Sku` อยู่ที่ `ProductVariant` ไม่ใช่ `Product`** → เลือกสินค้า 1 ตัว = หลาย SKU

**⚠️ 3rdparty-api ยังไม่ถูกผูกเข้า monorepo จริง:** `.gitmodules` มี entry แต่ยัง uncommitted และ `git ls-files -s backend/oc2plus-line-crm-service-3rdparty-api` = ว่าง (ไม่มี gitlink ใน index) → clone ใหม่จะไม่ได้ repo นี้

**รอบแก้ 2026-08-07 (5 สายขนาน) — แตกการ์ดเพิ่ม 5 ใบ:** **OC-4420** Commit layer (แยกจาก 4413 ที่เหลือ Evaluate; interface `AwardPlan → AwardResult`, "Commit ลดได้ ห้ามเพิ่มเกิน plan") · **OC-4421** ขอบเขตสินค้า+snapshot (แยกจาก 4295) · **OC-4422** QR โหมด preload lot (แยกจาก 4335 ที่เหลือ live-mint) · **OC-4423** attribution filter/ลิงก์พิเศษ (แยกจาก 4297 ที่เหลือ scenario 1-2) · **PAT-2605** on-demand order lookup (pull — คู่กับ PAT-2604 ที่เป็น push และระบุ scope ตัวเองว่าไม่รวม lookup)

**PO review 2026-08-07 (9 ใบ): ผ่านแค่ OC-4414 — แก้ครบทั้ง 5 รากแล้ว.** 5 รากปัญหา — decision ค้างในคอมเมนต์ไม่ merge เข้า description (4295 หยุดที่ A20 ทั้งที่เคาะถึง A28) · ชื่อ field ไม่ตรงกัน · **ไม่มี Keto authz AC เลยสักใบ** · 4295/4413/4335 ใหญ่เกิน sprint ต้องแตก · 4362/4407 ส่ง line_items ไม่ได้ (เครื่องจะ skip campaign เฉพาะสินค้า → ต้องเตือนใน UX + preview ต้องเรียก Evaluate จริงไม่ใช่คำนวณเอง)

⚠️ Jira link เดิมหลายเส้น**ทิศกลับ** (บันทึกเป็นใบลูก blocks ใบแม่) — เพิ่มเส้นถูกทิศแล้ว แต่ MCP ไม่มี deleteIssueLink ต้องลบของเก่าด้วยมือ

ดู [[project_loyalty_point_cluster]] · [[feedback_oc_pat_board_ownership_rule]] · artifact สรุป: https://claude.ai/code/artifact/f31cc7c7-9a85-4931-93a5-272e80e4f69e
