---
name: project_plan_capability_quota_anchor
description: "Anchor decision (2026-07-11): Commercial Plan (management-backend) = แกนเดียวของ monetization; Capability (boolean gate) + Quota (metered) เป็นสอง projection; ไม่สร้าง sellsuki-entitlement-service; glossary องค์กร 3 คำ"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

**Debate + PO เคาะ 2026-07-11 (BOLA-227 vs OC-4257 ชนกันเรื่องคำว่า "plan"):**

**Anchor = Commercial Plan เป็นแกนเดียว** — ตาราง `plan`/`plan_assignments` ใน management-backend (สร้าง Sprint 125, atomic+idempotent) คือ source of truth ว่า tenant ซื้ออะไร · มีสอง projection:
- **Capability** (boolean feature gate) → v2 เพิ่ม `capabilities[]` ใน plan → publish `plan.activated {tenant, capabilities}` ตอน assign → product cache local (BOLA `workspaces.tier`)
- **Quota** (metered allowance) → QMS AssignPlan ตามที่มี

**Glossary องค์กร (บังคับทุกบอร์ด):** Plan = SKU ที่ขาย · Capability = สิทธิ์เปิดฟีเจอร์ · Quota = โควตาใช้งาน — ห้ามใช้ "plan" หมายถึง tier

**ผลต่อ BOLA-227 (comment ลงแล้ว):** ❌ ไม่สร้าง `sellsuki-entitlement-service` (BOLA-232 เปลี่ยน scope) · rename `workspaces.plan` → `workspaces.tier` ก่อนเขียนโค้ด · BOLA-228 middleware/BOLA-237 consumer/BOLA-229 license key (self-hosted offline) คงเดิม · SukiPay → เรียก assign endpoint เดิม (ประสาน PAT-2493) · **Registration Forms = core ทุก tier ห้าม gate** (ไม่งั้นชน OC-4245/4289 chain)

Links: BOLA-227 ↔ OC-4257 relates แล้ว · ต่อยอดจาก [[project_quota_not_feature_gate]] (quota ≠ feature gate — ยังจริง, ตอนนี้มีคำตอบแล้วว่า gate ใช้ capability จาก plan) · [[project_qms_ui]]
