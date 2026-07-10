---
name: project_oms2_decouple_decision
description: OMS 2.0 — ทีมเลือก Decouple (PAT-2540) เป็น spec หลัก; โครง card cluster + children ใหม่ 2546-2550
metadata: 
  node_type: memory
  type: project
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

ทีมโหวตเลือก **Decouple approach (PAT-2540)** เป็นแนวทาง OMS 2.0 Order Flow & Fulfillment (2026-07-07) — order state **derive จาก task states** (ไม่ใช้ composite `payment:delivery` เป็น source of truth), fulfillment = pluggable task template, flow_type คงเป็น compat label ที่ derive จาก attribute

**Order state chain (rename 2026-07-10, 7 states):** `created → confirmed → preparing → ready_to_deliver → delivering → delivered → completed` — ตระกูล deliver เป็นคำกลาง physical/service/digital; **ห้ามใช้ `fulfilled`** (ชน WMS = แพ็ค/ส่งออกเสร็จ) และ order-level `processing` (ชน task status ใน code); `ready_to_deliver` = เตรียมเสร็จรออีกฝ่าย (packed รอขนส่งเข้ารับ / รอลูกค้ามารับ / พร้อมเข้ารับบริการ); shape ที่ไม่มีจังหวะไหน auto-pass. Interactive walkthrough สำหรับ dev: artifact claude.ai/code/artifact/82eab58a-3d26-41e2-a338-d62dbeb24037

**โครง Jira:** PAT-2540 = epic หลัก (PAT-2541 composite = reference ไม่ implement; PAT-2238/2239 = superseded shell) · children: PAT-2292/2293 (slices, consumer), 2296 (migration), 2297 (enum SoT — ขยาย scope activate DIGITAL_DELIVERY + SERVICE แล้ว), 2300, 2251, 2252, 2260, 2304 + **ใหม่ 5 ใบ**: PAT-2547 (derivation layer), 2548 (task-set generator), 2549 (task templates digital/service — MVP ไม่แตะ physical stock, license/capacity = future), 2550 (guard evaluator 2×2), 2546 (FE PaymentInfo.svelte เลิกอิง saleChannel)

**Dependency:** 2547/2548/2550 blocks 2292+2293 · 2297 blocks 2549 · generator/derivation นิยามอยู่ที่ 2548/2547 เท่านั้น (slices เป็น consumer — de-dup แล้วทั้ง 2292/2293)

**Source of truth:** `docs/oms2-study/18-flow-archetypes-guard-model.md` + `ship-flow.decouple.example.json` (repo, committed) — เกี่ยวกับ [[project_sukipay_void_rename]] (vocab COMPENSATING/COMPENSATED ที่ใช้ใน cluster นี้)
