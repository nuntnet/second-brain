---
name: project_oc2plus_consent_enforcement_model
description: OC2Plus consent pipeline — enforcement decoupled from type into a 3-layer model
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-03T04:50:59.684Z
---

Consent doc pipeline ของ CCS3 (OC-3897 = doc mgmt, OC-4089 = binding, OC-4340 = member accept — **ทั้งหมดยัง To Do, ยังไม่มีโค้ด**). Backend = `sellsuki-service-consent` (internal, ไม่มี auth, CCS3 เรียกผ่าน proxy).

**การตัดสินใจ 2026-08-03 (OC-3897): แยก enforcement ออกจาก type เป็น 3 เลเยอร์**
1. **Type (5 closed: pdpa/tos/marketing/cookie/privacy_notice)** = *legal envelope* — จำกัดว่าเลือกโหมดไหนได้ (marketing ห้าม require, privacy_notice ได้แค่ display)
2. **เอกสารมี field `enforcement` (default)** = radio 3 โหมด: `require_accept` (บล็อก submit + consentee) / `acknowledge_optional` (โชว์ ไม่บล็อก) / `display_only` (ลิงก์อ่าน ไม่มี consentee). เก็บเป็น tag `enforce:<mode>` (additive)
3. **Binding ต่อ surface (OC-4089)** = flow จริง (register/create-company) เลือกเอกสาร+โหมด (default จาก doc, override ได้แต่ห้ามเกิน envelope)

**Server-side consent gate (สำคัญ):** endpoint ที่จบ flow ต้อง verify ว่ามี consentee ของ required docs (ตรง version live) ครบก่อนสำเร็จ — ไม่เชื่อ client (กัน bypass). version ใหม่ → บังคับ renew.

Draft model: draft เก็บใน consent service (collection ใหม่, unique(owner_id, consent_id)), ไม่มีเลข version, version เกิดตอน publish เท่านั้น + live ทันที. Draft API 5 endpoints เป็นงาน backend additive.

ยังต้องต่อ: ไปอัปเดต OC-4089 ให้รับ enforcement-per-surface + registration/create-company wiring (hand-off ระบุใน OC-3897 Dependencies/DoD แล้ว).
