# ADAM SPS — Database Schema

**Source:** Code analysis + SQL queries from controllers
**Updated:** 2026-05-31

---

## ER Diagram — Core Entities

```mermaid
erDiagram
    BRANCH {
        int branch_id PK
        string branch_desc
        string branch_name
        string branch_code
        string branch_brand
        int dea_id
        int sto_br_id
    }

    USER {
        int u_id PK
        string u_name
    }

    ADAM_VEHICLE_OWNER {
        string vehicle_owner_code PK
        string cus_name
        string cus_homephone
        string cus_address
    }

    ADAM_VEHICLE {
        string vehicle_code PK
        string vehicle_registration
        string vehicle_owner_code FK
    }

    ADAM_APPOINTMENT {
        string appointment_code PK
        datetime appointment_date
        datetime appointment_timestamp
        int appointment_state
        int appointment_status
        string vehicle_owner_code FK
        string cus_name
        int branch_id FK
        int user_id FK
        int user_id_approver FK
    }

    ADAM_JOB {
        string job_code PK
        datetime job_draft
        datetime job_timestamp
        int job_status
        string vehicle_owner_code FK
        string cus_name
        int branch_id FK
        int user_id FK
    }

    ADAM_JOBBP {
        string job_code PK
        datetime job_draft
        datetime job_timestamp
        int job_status
        string vehicle_owner_code FK
        string cus_name
        int branch_id FK
    }

    ADAM_TASK {
        string task_code PK
        string task_reference FK
    }

    ADAM_TASK_ITEMS {
        string task_item_code PK
        string task_code FK
        int task_item_reference FK
        int task_item_booking
    }

    ADAM_SALE_ORDER {
        string sale_order_code PK
        datetime sale_order_draft
        datetime sale_order_timestamp
        int sale_order_status
        string cus_name
        int branch_id FK
        int user_id FK
    }

    ADAM_SALE_ORDER_ITEMS {
        string sale_order_item_code PK
        string sale_order_code FK
        int spare_id FK
        int sale_order_item_booking
    }

    ADAM_MAZDA_SPARE {
        int spare_id PK
        string spare_code
        string spare_barcode
        string spare_name_th
        string spare_name_en
        decimal spare_cost_price
        decimal spare_sale_price
        int spare_status
    }

    ADAM_TAX_INVOICE {
        string tax_invoice_code PK
        string tax_invoice_status
        string tax_invoice_item_reference_to
        string tax_invoice_item_reference
        int branch_id FK
    }

    BRANCH ||--o{ ADAM_APPOINTMENT : "branch_id"
    BRANCH ||--o{ ADAM_JOB : "branch_id"
    BRANCH ||--o{ ADAM_SALE_ORDER : "branch_id"
    ADAM_VEHICLE_OWNER ||--o{ ADAM_VEHICLE : "vehicle_owner_code"
    ADAM_VEHICLE_OWNER ||--o{ ADAM_APPOINTMENT : "vehicle_owner_code"
    ADAM_VEHICLE_OWNER ||--o{ ADAM_JOB : "vehicle_owner_code"
    ADAM_JOB ||--o{ ADAM_TASK : "job_code = task_reference"
    ADAM_JOBBP ||--o{ ADAM_TASK : "job_code = task_reference"
    ADAM_TASK ||--o{ ADAM_TASK_ITEMS : "task_code"
    ADAM_TASK_ITEMS }o--|| ADAM_MAZDA_SPARE : "task_item_reference = spare_id"
    ADAM_SALE_ORDER ||--o{ ADAM_SALE_ORDER_ITEMS : "sale_order_code"
    ADAM_SALE_ORDER_ITEMS }o--|| ADAM_MAZDA_SPARE : "spare_id"
    ADAM_JOB ||--o{ ADAM_TAX_INVOICE : "via reference"
    ADAM_SALE_ORDER ||--o{ ADAM_TAX_INVOICE : "via reference"
```

---

## Module Dependency Graph

```mermaid
graph LR
    subgraph Input
        AP[Appointment<br/>adam_appointment]
        SO[Sale Order<br/>adam_sale_order]
    end

    subgraph Workshop
        JB[Job / RO<br/>adam_job]
        BP[Body Paint<br/>adam_jobbp]
    end

    subgraph Parts
        SP[(Spare Parts<br/>adam_brand_spare)]
        ST[(Stock<br/>adam_stock)]
    end

    subgraph Output
        TI[Tax Invoice]
        RC[Receipt]
        CN[Credit Note]
    end

    AP -->|สร้าง| JB
    AP -->|สร้าง| BP
    SO --> TI
    JB --> TI
    JB --> RC
    BP --> TI
    SP --> ST
    JB -->|ใช้| SP
    SO -->|จอง| SP
    TI --> CN
```

---

## Document Number Formats

| Document | Format | Example |
|----------|--------|---------|
| Appointment | AP{YY}{MM}{XXXX} | AP260500001 |
| Job / RO | JB{YY}{MM}{XXXX} | JB2605001 |
| Body Paint | BP{YY}{MM}{XXXX} | BP2605001 |
| Sale Order | SO... | SO2605001 |
| Task | TA{YY}{MM}{XXXX} | TA2605001 |

**Note:** YY = 2 หลักสุดท้ายของปี CE (ไม่ใช่ พ.ศ.)

---

## Tables ที่ Dynamic (per branch / per brand)

### Per Brand
| Pattern | ตัวอย่าง |
|---------|---------|
| `adam_{brand}_spare` | adam_mazda_spare, adam_ford_spare |

### Per Branch  
| Pattern | ตัวอย่าง |
|---------|---------|
| `adam_{branch_id}_service` | adam_1_service |
| `adam_{branch_id}_config` | adam_1_config |
| `adam_{branch_id}_job_set` | adam_1_job_set |

### Branch → Brand Mapping (hardcoded)
```
branch_id 1, 2, 5 → adam_mazda_spare
branch_id 3       → adam_ford_spare
```

---

## Status Codes

### Appointment State
| State | Meaning |
|-------|---------|
| 1 | สร้างใหม่ |
| 6 | แก้ไข / re-appointment |
| 7 | อนุมัติแล้ว |

### Generic Status
| Value | Meaning |
|-------|---------|
| 0 | ลบแล้ว / Inactive |
| 1 | Active |

---

## Related Notes
- [[00-Overview]] — ภาพรวม
- [[01-Module-Detail]] — รายละเอียด controllers
- [[02-Integration-Plan]] — แผน integrate
