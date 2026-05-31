# ADAM SPS — Integration Plan กับ ch-erawan-next

**Goal:** ดึงข้อมูล ยอดจองรายวัน จาก ADAM SPS → แสดงใน ch-erawan-next dashboard
**Updated:** 2026-05-31

---

## สิ่งที่ต้องการ

**ยอดจองรายวัน** = จำนวน / รายการ Appointment ที่นัดในแต่ละวัน
- ดึงจาก `adam_appointment`
- Filter: วันที่ + branch_id
- แสดงเป็น graph รายวัน / รายสัปดาห์ / รายเดือน

---

## SQL ที่ต้องใช้

### ยอดจองรายวัน (simple)
```sql
SELECT
  DATE(appointment_date) AS booking_date,
  branch_id,
  COUNT(*) AS total_bookings,
  SUM(CASE WHEN appointment_state = 7 THEN 1 ELSE 0 END) AS approved_bookings
FROM adam_appointment
WHERE
  appointment_status = 1
  AND appointment_date BETWEEN '{start_date}' AND '{end_date}'
GROUP BY DATE(appointment_date), branch_id
ORDER BY booking_date ASC
```

### ยอดจองรายวัน (พร้อม customer info)
```sql
SELECT
  appointment_code,
  DATE(appointment_date) AS booking_date,
  appointment_date,
  appointment_state,
  cus_name,
  cus_homephone,
  branch_id,
  user_id,
  appointment_timestamp
FROM adam_appointment
WHERE
  appointment_status = 1
  AND DATE(appointment_date) = '{target_date}'
  AND branch_id = {branch_id}
ORDER BY appointment_date ASC
```

---

## 3 Options เปรียบเทียบ

### Option A — Direct MySQL Query (from Next.js)

```
ch-erawan-next → mysql2 driver → ADAM MySQL DB
```

**ข้อดี:**
- ไม่ต้องแก้ ADAM SPS เลย
- Query ได้ทันที
- Real-time data

**ข้อเสีย:**
- ต้องเปิด MySQL port หรือ tunnel ผ่าน Docker network
- Security risk ถ้า expose ออก internet
- Coupling โดยตรงกับ schema ของ ADAM

**วิธีทำ:**
```ts
// lib/adam-db.ts
import mysql from 'mysql2/promise'

const pool = mysql.createPool({
  host: process.env.ADAM_DB_HOST,
  user: process.env.ADAM_DB_USER,
  password: process.env.ADAM_DB_PASS,
  database: process.env.ADAM_DB_NAME,
})

export async function getDailyBookings(date: string, branchId: number) {
  const [rows] = await pool.query(
    `SELECT DATE(appointment_date) AS booking_date,
            COUNT(*) AS total
     FROM adam_appointment
     WHERE appointment_status = 1
       AND DATE(appointment_date) = ?
       AND branch_id = ?
     GROUP BY DATE(appointment_date)`,
    [date, branchId]
  )
  return rows
}
```

---

### Option B — Thin PHP API Layer (บน ADAM SPS)

เพิ่ม route ใน `routes.php` ของ ADAM SPS:

```php
// application/routes.php
Route::get('api/v1/daily-bookings', function() {
  $date   = Input::get('date', date('Y-m-d'));
  $branch = Input::get('branch_id', '');
  
  $where = "appointment_status = 1 AND DATE(appointment_date) = '{$date}'";
  if ($branch) $where .= " AND branch_id = {$branch}";
  
  $rows = DB::query("SELECT ... FROM adam_appointment WHERE {$where}");
  
  return Response::json(['data' => $rows, 'date' => $date]);
});
```

**ข้อดี:**
- API contract ชัดเจน
- ADAM เป็น data owner
- ง่ายต่อการ auth ในอนาคต

**ข้อเสีย:**
- ต้องแก้ ADAM SPS (deploy ใหม่)
- ต้อง expose HTTP port ออก internet หรือ VPN

---

### Option C — Cron Export → Turso (แนะนำ)

```
ADAM MySQL → Cron script (Node/PHP) → Turso (SQLite) → ch-erawan-next reads
```

**Flow:**
1. Cron ทุกชั่วโมง (หรือทุกคืน): query `adam_appointment` → upsert ลง Turso
2. ch-erawan-next: query Turso ด้วย Drizzle ORM เหมือนปกติ
3. Data อาจ delay สูงสุด 1 ชั่วโมง (ok สำหรับ daily report)

**ข้อดี:**
- ch-erawan-next ไม่ต้องรู้จัก MySQL ของ ADAM
- ใช้ Turso ที่มีอยู่แล้ว
- ไม่ต้องแก้ ADAM SPS
- Resilient — ถ้า ADAM down, ch-erawan ยังดึงข้อมูลล่าสุดได้

**ข้อเสีย:**
- Delay (not real-time)
- ต้องดูแล cron script

**Cron script (Node.js):**
```ts
// scripts/sync-adam-bookings.ts
import mysql from 'mysql2/promise'
import { createClient } from '@libsql/client'

const adam = mysql.createPool({ ... })
const turso = createClient({ url: process.env.TURSO_URL, authToken: process.env.TURSO_TOKEN })

async function sync() {
  const today = new Date().toISOString().split('T')[0]
  
  const [rows] = await adam.query<any[]>(
    `SELECT appointment_code, appointment_date, cus_name, 
            branch_id, appointment_state, appointment_status
     FROM adam_appointment
     WHERE DATE(appointment_date) >= DATE_SUB(?, INTERVAL 7 DAY)`,
    [today]
  )
  
  for (const row of rows) {
    await turso.execute({
      sql: `INSERT OR REPLACE INTO adam_appointments 
            (code, date, cus_name, branch_id, state, status)
            VALUES (?, ?, ?, ?, ?, ?)`,
      args: [row.appointment_code, row.appointment_date, 
             row.cus_name, row.branch_id, row.appointment_state, row.appointment_status]
    })
  }
  
  console.log(`Synced ${rows.length} bookings`)
}

sync()
```

**Turso schema (Drizzle):**
```ts
// db/schema/adam-bookings.ts
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core'

export const adamAppointments = sqliteTable('adam_appointments', {
  code: text('code').primaryKey(),
  date: text('date').notNull(),
  cusName: text('cus_name'),
  branchId: integer('branch_id'),
  state: integer('state'),
  status: integer('status'),
  syncedAt: text('synced_at').default(sql`CURRENT_TIMESTAMP`),
})
```

---

## คำแนะนำ

**เริ่มด้วย Option A** ก่อน (direct MySQL) เพื่อ validate data ได้เร็ว
ถ้า ok ค่อยย้ายไป Option C สำหรับ production

### Next Steps
- [ ] ตรวจสอบ network path: Docker network ของ ADAM → ch-erawan-next
- [ ] ดู `.env` ของ ADAM สำหรับ DB credentials
- [ ] สร้าง `adam_appointments` table ใน Turso
- [ ] เขียน cron sync script
- [ ] สร้าง `/dashboard/bookings` page ใน ch-erawan-next

---

## Related Notes
- [[00-Overview]] — ภาพรวม ADAM SPS
- [[01-Module-Detail]] — SQL query จริงๆ ของ appointment module
- [[03-Database-Schema]] — ER diagram
