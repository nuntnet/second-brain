---
name: project-oms2-plan-gaps-2026q3
description: "Full gap analysis 2026-07-03 — OMS 2.0 ไม่มี SP สักใบ, 2244↔2057 lock เป็น fiction, RAG P0 ไม่มี epic, parcel 19 ใบ superseded แต่ยังนับใน 2241"
metadata: 
  node_type: memory
  type: project
  originSessionId: b0b4a24f-96f9-41ef-b15d-18e86ad938e1
---

ผล full gap analysis ของแผนพัฒนา (po-team, 2026-07-03) — verified กับ Jira live:

**Critical:**
- OMS 2.0 cluster (~60 cards: PAT-2238/2239 children + LINK_BILL 21 + COD chain) **ไม่มี story point สักใบ** → wave→sprint mapping S100–103 เป็น dependency ordering ไม่ใช่ delivery schedule จนกว่าจะ estimate. Contrast: SukiPay ops S95–97 มี SP ครบ (21 SP)
- Cross-cluster lock `PAT-2244 ↔ PAT-2057` ที่ epic-index อ้าง **ไม่มี issuelink จริงใน Jira** ทั้งสองฝั่ง — ไม่มีอะไร enforce
- `PAT-2057` เป็น epic เปลือก — children หลัก (2047 ABSORBED / 2048 REDUNDANT), description ยังอ้าง S93–94; ต้อง re-scope ก่อนเริ่ม PAT-2244
- `PAT-2294` (LINK_BILL) sprint ขัดกันเอง: description บอก S101, epic-index บอก S102–103; labels ยัง Draft; 21 children unestimated
- **RAG เป็น P0 pillar แต่ไม่มี epic** — vision จริงมี 12 themes ใน data.js (roadmap.md เขียนแค่ 9); spike PAT-2473 ลอยเป็น orphan รอบ้าน. Sale Channel Listing Matrix (P0) ก็ไม่มีบ้าน (PAT-2494 ครอบแค่ pricing/bundle)
- Parcel batch 19 ใบถูก re-parent เข้า PAT-2241 ทั้งที่ codegraph บอก superseded โดย PAT-2250 → rollup PAT-2241 ผิดถาวรจนกว่าจะตัดสินปิด

**Backlog hygiene ค้าง:** 51 STALE orphans แนะนำปิดแต่ไม่ execute (AMS-v2 ต้อง verify Kratos ก่อน), 21 [Technical] orphans รู้บ้านแล้ว (PAT-2489) แต่ยังไม่ attach

เกี่ยวข้อง [[project-pat-epic-links-unwired]] [[project-sukipay-refund-cluster]]
