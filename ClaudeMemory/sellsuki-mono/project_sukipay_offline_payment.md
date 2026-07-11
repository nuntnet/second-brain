---
name: project_sukipay_offline_payment
description: SukiPay offline payment (bankTransferOffline) wire ครบ end-to-end แล้ว — no-slip instant COMPLETED; PAT-2034 online-only by design; correction = void PAT-2441 + adjust PAT-2288
metadata: 
  node_type: memory
  type: project
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

SukiPay **offline payment path wire ครบ end-to-end จริง** (verified จาก code 2026-07-11) — แก้ความเข้าใจผิดว่า "ยังไม่ได้จัดการ offline":

**Wire chain (file:line):** POS → OMS `order_payment_service.CreatePayment(method string)` → `uc.CreatePayment` → `sukiPayPaymentRepository.ProcessPayment` (OMS `src/use_case/payment.go:39`) → `toPaymentMethod` map `bankTransferOffline`→pb enum (OMS `src/repository/sukipay_payment_repository/grpc_helper.go:14`) → SukiPay gRPC `ProcessPayment` → `parsePaymentMethod` (sukipay `src/use_case/payment.go:214`) → `newPaymentProcessor` → `bankTransferOfflineProcessor` (sukipay `src/use_case/payment_processor.go:36,229-310`)

**Offline processor behavior:** slip OPTIONAL (`createSlipIfPresent`, ไม่มี `ErrSlipURLRequired` — ต่างจาก `bankTransferProcessor:149` ที่บังคับ slip) · set state ตรงเป็น `COMPLETED` (line 281) ข้าม UNDER_REVIEW · settle ทันที (`instantApplyToTransaction`) · require แค่ BankCode+TransferAt+active channel · `cashProcessor` ก็ instant COMPLETED. method = passthrough string จาก POS caller (`bankTransferOffline`)

**PAT-2034 (Done) เป็น online-only BY DESIGN** — approve/reject/edit guard `StateUnderReview` (sukipay `entity/payment/payment.go:71,81`, `use_case/payment_edit.go:75`) → offline ที่ไป COMPLETED ตรงใช้ไม่ได้ **และไม่ต้องใช้** เพราะ cashier = reviewer เอง (ยืนยันยอดที่ POS ไม่มี slip ให้ review)

**Correction ของ offline COMPLETED = ใช้ของเดิม (user ยืนยัน 2026-07-11):** void = PAT-2441 (COMPLETED→VOIDED, tx→PENDING, cash+bank) ใต้ epic PAT-2440 · adjust ยอด = PAT-2288 · แต่ `payment.Void()` method **ยังไม่มีใน entity** (มีแค่ Approve/Reject) → PAT-2441 ยัง To Do จริง (ยังไม่ build). Related: OMS offline checkout = PAT-2208/2213 (Done), cash = PAT-1947 (Done)

เกี่ยวกับ [[project_sukipay_void_rename]] · [[project_oms2_decouple_decision]] (offline instant = A1/A6 ใน 2×2)
