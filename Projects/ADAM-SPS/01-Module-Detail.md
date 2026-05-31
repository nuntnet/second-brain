# ADAM SPS — Module Detail

**Source:** codegraph index + direct code read
**Updated:** 2026-05-31

---

## Appointment (นัดหมาย)

**Controller:** `laravel/application/controllers/appointment.php`
**Table:** `adam_appointment`

### Methods
| Method | Route | Description |
|--------|-------|-------------|
| `get_form` | GET /appointment/form | แสดงฟอร์มนัดหมาย |
| `post_index` | POST /appointment | สร้างนัดหมายใหม่ |
| `put_index` | PUT /appointment | แก้ไขนัดหมาย |
| `delete_index` | DELETE /appointment | ลบนัดหมาย |
| `put_state` | PUT /appointment/state | เปลี่ยน state |
| `get_recent` | GET /appointment/recent | ดูนัดหมายล่าสุด |
| `get_search` | GET /appointment/search | ค้นหา |
| `get_manhour` | GET /appointment/manhour | ดู man-hour |
| `put_print` | PUT /appointment/print | พิมพ์ใบนัด |

### Appointment Code Format
```
AP{YY}{MM}{XXXX}
ตัวอย่าง: AP2605001 = พฤษภาคม 2569 ลำดับที่ 001
```

### Key Fields (adam_appointment)
```sql
appointment_code       VARCHAR  -- AP250500001
appointment_date       DATETIME -- วันที่นัด
appointment_timestamp  DATETIME -- วันที่สร้าง (CE)
appointment_state      INT      -- state ของการนัด
appointment_status     INT      -- 1=active, 0=deleted
appointment_note       TEXT     -- หมายเหตุ
appointment_tax_rate   DECIMAL  -- อัตราภาษี
appointment_tax_type   VARCHAR
vehicle_owner_code     VARCHAR  -- FK → adam_vehicle_owner
cus_name               VARCHAR  -- ชื่อลูกค้า
cus_address            TEXT
cus_homephone          VARCHAR
branch_id              INT      -- FK → branch
user_id                INT      -- FK → user (ผู้บันทึก)
user_id_approver       INT      -- FK → user (ผู้อนุมัติ)
appointment_approve    DATETIME -- วันที่อนุมัติ
job_code_before        INT      -- อ้างอิง job ก่อนหน้า
```

### State Flow
```
State 1 = สร้างใหม่
State 6 = แก้ไข (re-appointment)
State 7 = อนุมัติ (auto ถ้ามี permission 7,3 + 7,6)
```

---

## Booking Card (ใบจอง Spare Parts)

> ⚠️ นี่คือการจอง **อะไหล่** ไม่ใช่จองรถ

**Controller:** `laravel/application/controllers/booking_card.php`

### SQL Query (get_datatoshow) — ยอดจองอะไหล่รายวัน

```sql
-- UNION 3 sources: Sale Order, Job (RO), Body Paint
SELECT
  sale_order_item_code AS num,
  spare_code, spare_barcode, spare_detail, spare_name_th,
  sale_order_item_booking AS booking,
  CASE WHEN sale_order_timestamp IS NULL 
    THEN sale_order_draft 
    ELSE sale_order_timestamp 
  END AS log_time,
  s.sale_order_code AS code,
  cus_name
FROM adam_sale_order s
JOIN adam_sale_order_items sit ON s.sale_order_code = sit.sale_order_code
JOIN {adam_brand_spare} sp ON sit.spare_id = sp.spare_id
WHERE branch_id = {branch_id}
  AND spare_code LIKE '%{search}%'
  AND sale_order_status = 1
  AND sale_order_item_booking > 0
  AND (sale_order_draft OR sale_order_timestamp) BETWEEN {start} AND {end}

UNION

-- Job (RO) bookings
SELECT task_item_code, ..., j.job_code, cus_name
FROM adam_job j
JOIN adam_task t ON j.job_code = t.task_reference
JOIN adam_task_items tit ON t.task_code = tit.task_code
JOIN {adam_brand_spare} sp ON tit.task_item_reference = sp.spare_id
WHERE ... job_status = 1 AND task_item_booking > 0

UNION

-- Body Paint bookings
SELECT task_item_code, ..., jbp.job_code, cus_name
FROM adam_jobbp jbp
JOIN adam_task t ON jbp.job_code = t.task_reference
...

GROUP BY spare_code
ORDER BY spare_code ASC
```

**Date conversion:** `year_BE - 543` ก่อนส่งเป็น WHERE clause

---

## Sale Order (SO)

**Controller:** `laravel/application/controllers/sale_order.php`
**Class:** `Sale_Order_Controller`

### Methods
| Method | Description |
|--------|-------------|
| `post_sale_order` | สร้าง SO ใหม่ |
| `put_sale_order` | แก้ไข SO |
| `get_deletesale_order` | ลบ SO |

### Relationships
```
adam_sale_order
  ├── adam_sale_order_items (1:N)
  ├── adam_tax_invoice (via reference)
  ├── invoice_sale_order (billing layer)
  └── billinginvoice_sale_order (billing)
```

---

## Job / Repair Order (JB)

**Controller:** `laravel/application/controllers/job.php`
**Table:** `adam_job`

### Document Flow
```
Appointment (AP...)
  └── Job / RO (JB...)
        ├── Task (TA...)
        │     └── Task Items (อะไหล่ + บริการ)
        ├── Tax Invoice
        ├── Receipt
        └── Close Job
```

### Related Tables
- `adam_job` — header
- `adam_task` — task ภายใน job
- `adam_task_items` — line items (อะไหล่/บริการ)
- `adam_vehicle_owner` — เจ้าของรถ
- `adam_vehicle` — ทะเบียน

---

## Body Paint Job (BP)

**Controller:** `laravel/application/controllers/jobbp.php`
**Table:** `adam_jobbp`

คล้าย Job แต่สำหรับงานพ่นสี/อุบัติเหตุ
- Code: `BP...`
- มี `jobbpstatus`, `jobbpupload` เป็น sub-controllers

---

## Reports

| Controller | Description |
|-----------|-------------|
| `report_job` | รายงาน RO (พิมพ์ Excel + graph) |
| `report_jobbp` | รายงาน Body Paint |
| `report_sale_order` | รายงาน Sale Order |
| `report_tax_invoice` | รายงานใบกำกับภาษี |
| `report_technician` | รายงานช่างแต่ละคน |
| `report_job_graph` | กราฟยอด RO |
| `report_sms_customer` | รายงาน SMS ลูกค้า |

**Output format:** PHP → PHPExcel → `.xlsx` download
**No JSON API** — render ตรงในฝั่ง server

---

## User & Auth

**Table:** `user`
**Class:** `Loginfo` (helper)

```php
// Permission check pattern
if (Loginfo::get_permiss(7, 3) == true && Loginfo::get_permiss(7, 6) == true) {
    // auto-approve
}
```

Permission system: 2-parameter (module_id, action_id)
