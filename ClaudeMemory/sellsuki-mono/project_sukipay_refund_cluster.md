---
name: project_sukipay_refund_cluster
description: PATONA SukiPay refund/payment card cluster — role ของแต่ละ card + recurring attribution bug + sync/async pattern
metadata: 
  node_type: memory
  type: project
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

PATONA refund/payment cluster (PAT) — role ที่ถูกต้องของแต่ละ card (จำไว้เพราะ **cards หลายใบ attribute ผิด ซ้ำ ๆ**):
- **PAT-2424** = RequestRefund RPC → สร้าง finance task + ดัน payment `COMPLETED→REFUND_PENDING` (= **producer** ของ refund)
- **PAT-2289** = Refund Execution → Finance confirm/reject โอนเงินจริง (`REFUND_PENDING→REFUNDED`); reject rollback→COMPLETED
- **PAT-2036** = Decision Queue = **read-only list UI เท่านั้น** ไม่ execute/produce refund
- **PAT-2480** = Edit Refund Request (split จาก 2424)
- **PAT-2478** = Webhook HTTP Delivery Infra (HMAC+retry) — gap: PAT-2050 emit Kafka เท่านั้น, PAT-2115 = subscription CRUD → ไม่มีใครยิง webhook HTTP จริง
- **PAT-2290** = Multi-Tender VOID SAGA (completion checker VOID_PREPARED→VOID) — ยกเลิกไม่ได้ แม้ cash-only void ก็ต้องใช้

⚠️ **Recurring bug: attribution ผิดใบ** — cards ชอบเขียน refund execution = PAT-2036 (ผิด). ทุกครั้งที่เจอ "refund"/"REFUND_PENDING" ใน card ให้ verify ว่าชี้ 2424 (produce) / 2289 (execute) ไม่ใช่ 2036. เจอใน PAT-2029/2289/2425/2038.

**Integration pattern (ground จาก code):** OMS→SukiPay = **sync gRPC command** (ProcessPayment/Cancel/RequestRefund ฯลฯ); SukiPay→OMS = **async Kafka event** (worker_sukipay_transaction_event). `overpay_delta` ไม่มี Kafka producer — เป็น **derived state** ที่ SukiPay คำนวณจาก net_amount (OMS PATCH sync) เทียบ sum(COMPLETED).

**3 state layers (อย่าปน):** OMS payment_status (lowercase: unpaid/partially_paid/paid) · SukiPay transaction state (UPPER: PENDING/CLOSED/SETTLED/VOID_PREPARED/VOID — ดู [[project_sukipay_void_rename]]) · payment state (VOIDED). `CLOSED` = จ่ายครบแล้ว (ไม่ใช่ open; open = PENDING).

**OC2Plus customer identity (PAT-2477):** OMS Buyer PartyInfo มีแค่ phone/email/name **ไม่มี LINE UID** → member resolution ใช้ phone→email fallback.
