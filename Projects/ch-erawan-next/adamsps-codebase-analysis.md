# ADAM SPS — Codebase Analysis (CodeGraph)

> Source: `/Users/nunt/adamsps/adam/laravel`
> Analyzed: 2026-05-31 via CodeGraph v0.9.7
> Purpose: Integration planning สำหรับ ch-erawan-next dashboard (ยอดจองรายวัน)

---

## Stack

- Framework: Laravel 3 (PHP) — server-side render, NO REST API
- DB: MySQL (multi-branch via `branch_id`)
- Calendar: Buddhist calendar (พ.ศ.) — date format `DD-MM-YYYY` พ.ศ.
- Docker: runs via docker-compose, env vars: `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`
- Path: `~/adamsps/adam/laravel/`

---

## CodeGraph Stats

| Metric | Value |
|--------|-------|
| Files indexed | 4,167 |
| Nodes | 19,265 |
| Edges | 66,930 |
| DB size | 117.9 MB |
| PHP files | 2,298 |
| JS files | 1,758 |

---

## Module Map (Controllers)

### Core Business Modules

| Controller | File | ขนาด | หน้าที่ |
|-----------|------|------|---------|
| `booking_card_Controller` | booking_card.php | 11KB | ใบจองรถ (booking) |
| `Sale_Order_Controller` | sale_order.php | 98KB | ใบสั่งขาย |
| `quotation_Controller` | quotation.php | 47KB | ใบเสนอราคา |
| `appointment_Controller` | appointment.php | 25KB | นัดหมาย |
| `customer_detail_Controller` | customer_detail.php | 30KB | ข้อมูลลูกค้า |

### Workshop / Job Modules

| Controller | File | ขนาด | หน้าที่ |
|-----------|------|------|---------|
| `job_Controller` | job.php | 129KB | งานซ่อม (RO) |
| `jobbp_Controller` | jobbp.php | 213KB | งานซ่อมตัวถัง/สี (BP) |
| `close_job_Controller` | close_job.php | 35KB | ปิดงานซ่อม |
| `close_jobbp_Controller` | close_jobbp.php | 45KB | ปิดงาน BP |

### Inventory / Parts

| Controller | File | ขนาด | หน้าที่ |
|-----------|------|------|---------|
| `spare_Controller` | spare.php | 63KB | อะไหล่ |
| `goods_receipt_Controller` | goods_receipt.php | 105KB | รับสินค้า/อะไหล่ |
| `stock_card_Controller` | stock_card.php | 74KB | stock card |
| `stock_balance_Controller` | stock_balance.php | 33KB | ยอดคงเหลือ |
| `purchase_order_Controller` | purchase_order.php | 61KB | ใบสั่งซื้อ |

### Finance / Invoice

| Controller | File | ขนาด | หน้าที่ |
|-----------|------|------|---------|
| `billinginvoice_Controller` | billinginvoice.php | 76KB | ใบแจ้งหนี้ |
| `invoice_sale_order_Controller` | invoice_sale_order.php | 165KB | ใบแจ้งหนี้ Sale Order |
| `tax_invoice_Controller` | tax_invoice.php | 41KB | ใบกำกับภาษี |
| `receipt_Controller` | receipt.php | 15KB | ใบเสร็จ |
| `credit_note_Controller` | credit_note.php | 28KB | ใบลดหนี้ |

### Report Modules

| Controller | File | ขนาด | หน้าที่ |
|-----------|------|------|---------|
| `report_sale_order_Controller` | report_sale_order.php | 122KB | รายงาน Sale Order |
| `report_job_Controller` | report_job.php | 71KB | รายงานงานซ่อม |
| `report_jobbp_Controller` | report_jobbp.php | 162KB | รายงาน BP |
| `report_jobbp_new_Controller` | report_jobbp_new.php | 152KB | รายงาน BP (ใหม่) |
| `report_job_claim_Controller` | report_job_claim.php | 123KB | รายงาน claim |
| `report_technician_Controller` | report_technician.php | 38KB | รายงานช่าง |
| `report_items_Controller` | report_items.php | 37KB | รายงานอะไหล่ |
| `report_tax_invoice_Controller` | report_tax_invoice.php | 83KB | รายงานใบกำกับ |

---

## Key Tables for ยอดจองรายวัน

### booking_approval (หลัก)

```
boo_st_id     → สถานะการจอง
                  4 = รอส่งมอบ
branch_id     → สาขา
cus_id_buy    → FK → customer table
sto_mo_id     → FK → stock_model (รุ่นรถ)
u_id_sale     → เซลส์
app_delivery  → วันส่งมอบ (วันที่)
```

### ตารางที่เกี่ยวข้อง

- `customer` — ข้อมูลลูกค้า (join via cus_id_buy)
- `stock_model` — รุ่นรถ (join via sto_mo_id)
- `branch` — สาขา (join via branch_id)
- `adam_sale_order` — ใบสั่งขาย
- `adam_sale_order_items` — รายการสินค้า

### Date Format Note

วันที่ใน ADAM เก็บแบบ Buddhist calendar (พ.ศ.) format `DD-MM-YYYY`
ต้องแปลงเมื่อ integrate กับ ch-erawan-next:

```php
// ADAM pattern (จาก source code)
$tmp = explode('-', $start_date);
$start_date = $tmp[2] . '-' . $tmp[1] .'-' . $tmp[0];
// แปลง year: พ.ศ. - 543 = ค.ศ.
```

---

## Integration Approach Options

### Option A — Direct MySQL Query
```
ch-erawan-next (Next.js) → MySQL (ADAM DB via Docker network)
```
- ✅ ไม่ต้องแก้ ADAM
- ✅ Real-time data
- ❌ Tight coupling กับ ADAM DB schema
- ❌ ต้องเปิด MySQL port ออกนอก Docker

### Option B — Thin PHP API Layer (แนะนำ)
```
ch-erawan-next → PHP endpoint ใหม่ใน ADAM → MySQL
```
- ✅ ADAM เป็น data owner
- ✅ ไม่ต้องเปิด MySQL port
- ✅ ทำ date conversion ใน PHP ได้
- ❌ ต้องเพิ่มไฟล์ใน ADAM codebase

### Option C — Cron Export to Turso
```
ADAM MySQL → cron export → Turso (SQLite) → ch-erawan-next
```
- ✅ ch-erawan-next ไม่ depend on ADAM uptime
- ✅ Fast queries (Turso edge)
- ❌ ข้อมูล delay ตาม cron interval
- ❌ ซับซ้อนที่สุด

---

## SQL Query ตัวอย่าง (ยอดจองรายวัน)

```sql
SELECT 
  DATE(ba.app_delivery) AS delivery_date,
  b.branch_desc AS branch_name,
  sm.model_name AS car_model,
  COUNT(*) AS booking_count,
  CONCAT(c.cus_firstname, ' ', c.cus_lastname) AS customer_name,
  ba.u_id_sale AS sale_staff
FROM booking_approval ba
LEFT JOIN branch b ON b.branch_id = ba.branch_id
LEFT JOIN stock_model sm ON sm.sto_mo_id = ba.sto_mo_id
LEFT JOIN customer c ON c.cus_id = ba.cus_id_buy
WHERE ba.boo_st_id = 4  -- รอส่งมอบ
  AND DATE(ba.app_delivery) = CURDATE()
GROUP BY ba.branch_id, sm.model_name
ORDER BY delivery_date DESC;
```

> ⚠️ ต้องตรวจสอบชื่อ column จริงจาก DB dump ก่อน — field names จากโค้ดอาจต่างจาก actual schema

---

## Next Steps

- [ ] ตรวจสอบ actual DB schema จาก MySQL dump หรือ `DESCRIBE booking_approval`
- [ ] ตัดสินใจ integration approach (A/B/C)
- [ ] สร้าง connection / API ใน ch-erawan-next
- [ ] Design dashboard UI สำหรับ ยอดจองรายวัน พร้อมกราฟ
- [ ] handle date conversion (พ.ศ. → ค.ศ.)

---

Tags: #adamsps #ch-erawan #dms #integration #booking-report #codegraph
