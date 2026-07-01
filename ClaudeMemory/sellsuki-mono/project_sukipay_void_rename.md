---
name: project_sukipay_void_rename
description: "SukiPay transaction state rename VOID_PREPARED→Compensating, VOID→Compensated — impact plan + locked decisions"
metadata: 
  node_type: memory
  type: project
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

SukiPay **transaction** state rename เพื่อเลิกสับสนกับ **payment** state `VOIDED` (คนละ level). ใช้ **UPPER CASE ทั้งหมด** ให้ตรง convention เดิม (PENDING/CLOSED/VOID_PREPARED):
- `VOID_PREPARED` → `COMPENSATING`
- `VOID` → `COMPENSATED`
- payment `VOIDED` **คงเดิม ไม่แตะ** ([payment.go:18](backend/sellsuki-pay-backend/src/entity/payment/payment.go:18))

**Decisions locked (2026-07):**
- ใช้ CAPS ทั้งหมด (COMPENSATING/COMPENSATED). Kafka event names เปลี่ยนด้วย (`VOID_PREPARED`/`TRANSACTION_VOIDED`) → breaking, ต้อง coordinated deploy SukiPay+OMS
- RPC/action names เปลี่ยนตาม (`InitiateVoid`→`InitiateCompensation` ฯลฯ) — RPCs ยังไม่ implement (backlog ใน epic-index.md)
- Outline + 6 unread cards (PAT-2097/2288/2447/2443/2049/2040): รอ Outline MCP กลับมาแล้วทำต่อ

**Code impact (sellsuki-pay):** enum [payment_transaction.go:22-23](backend/sellsuki-pay-backend/src/entity/payment_transaction/payment_transaction.go:22), state machine `payment_transaction.go:20-21`, events.go:37-38, list_with_payment.go:52, gorm check constraint postgres_gorm_model.go:23 + migration 0005. **DB = varchar+CHECK → data migration จำเป็น** (UPDATE rows + alter constraint). `transaction_audit_trails` jsonb append-only → historical เก่าคงชื่อเดิม backfill ไม่ได้. OMS: sukipay_event.go:53 (consumer, ยังไม่มีที่ใช้).

**Jira cards ต้องแก้ (transaction-level):** PAT-2426(In Progress), PAT-2290, PAT-2425, PAT-2286, PAT-2441(In Progress), PAT-2036. ไม่แตะ: PAT-2208/2289/2424 (payment-level/RPC). PAT-2426 มี "Compensated terminal" ในตารางอยู่แล้วแต่ state machine ยังใช้ VOID → rename แก้ inconsistency ในตัวเอง.

**Conflict:** OMS SAGA (PAT-2345) ใช้ "Compensating actions" = verb; state ใหม่ = noun → ไม่ชนใน code แต่สับสนในเอกสารที่พูดทั้ง 2 ระบบ (PAT-2290/2038).

รอบนี้ทำแค่ plan (read-only) ยังไม่แก้. เกี่ยวกับ [[project_control_tower]] refund cluster ที่ทำใน session เดียวกัน.
