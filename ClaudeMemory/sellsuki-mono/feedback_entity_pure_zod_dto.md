---
name: feedback-entity-pure-zod-dto
description: Senior review เคาะ — entity/domain layer ต้อง pure TS zero-dependency; zod อยู่ชั้น DTO/boundary เป็น z.ZodType<Entity>; อย่าเอา schema lib เป็นตัว model
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c107f1b5-8d84-48d1-b495-a8743dfcb35b
  modified: 2026-08-04T05:23:39.590Z
---

**Senior review บน frontend-kit design (2026-08-04):** "ไม่แนะนำเอา Zod เป็น entity — Zod ควรอยู่ level DTO/Interface Layer, Entity ควร pure ไม่ extend class+library ข้างนอก" — ผมเคยออกแบบให้ zod schema เป็นตัว domain model (z.infer เป็น type) ซึ่งผิดหลัก clean architecture

**Why:** ชั้นในสุดต้อง zero-dependency — library ใดๆ (zod, neverthrow, ORM) ที่เข้าไปอยู่ใน entity ทำให้ชั้นที่ควรเสถียรสุดสั่นตาม lib churn และผูก domain กับ mechanism

**How to apply:**
- Entity = pure TS interface + pure function เสมอ (ทั้ง frontend และ Go — ฝั่ง Go boilerplate ก็หลักเดียวกัน: entity ไม่รู้จัก GORM tag ในอุดมคติ)
- Validation schema อยู่ชั้น boundary/DTO ประกาศ `z.ZodType<Entity>` — TypeScript บังคับ schema ตรง entity (drift = compile error) และทิศ dependency ชี้เข้าใน
- Pattern = "parse, don't validate": wire/unknown → parse ที่ boundary → คืน pure entity
- Utility type อย่าง Result ถ้าจะอยู่ foundation ให้เขียนเอง (~60 บรรทัด) ไม่ import lib
- บังคับด้วยเครื่องมือ: dependency-cruiser rule ห้าม foundation/domain import npm package ใดๆ
- ใช้กับ [[ai-chat-platform-plan]] frontend-kit (PAT-2587) แล้ว — จำไว้ใช้กับ boilerplate/kit อื่นทุกตัวในอนาคต
