---
name: feedback_user_runs_codex_in_parallel
description: user เดินงานคู่ขนานด้วย ChatGPT/Codex บนรีโปเดียวกัน (branch ขึ้นต้น codex/*) — ต้อง re-establish state ทุกครั้งที่กลับมา และห้ามแก้ไฟล์ที่อีกฝั่งมีงานค้างยังไม่ commit
metadata: 
  node_type: memory
  type: feedback
  originSessionId: fa22ebb1-f715-4667-85a1-61ebbcc816ab
  modified: 2026-09-12T22:43:36.238Z
---

user บอกเองเมื่อ 2026-09-13: *"มี chatgpt ช่วยทำต่อไปเยอะละ"* — งาน OC2Plus
customer app เดินคู่ขนานระหว่างผมกับ ChatGPT/Codex บนรีโปชุดเดียวกัน

**Why:** ไม่ใช่การทำงานคนเดียวแล้ว · รอบนั้นทั้ง `frontend/oc2plus-linecrm-frontend-member`
และ `backend/oc2plus-line-crm-service-member-api` ถูกย้ายไปอยู่บน branch
**`codex/oc-4344-customer-app-integration`** โดยที่ผมไม่รู้ · ภาพที่ผมจำไว้จาก
เทิร์นก่อนหน้า (branch, commit, ไฟล์) **ใช้ไม่ได้แล้ว** และ working tree มีงาน
ค้างที่ยังไม่ commit ของอีกฝั่งอยู่ด้วย

**How to apply:**
1. **เริ่มเทิร์นด้วยการเช็คของจริงก่อนเสมอ** — `git branch --show-current`,
   `git log --oneline -8`, `git status --short` ทั้งสองรีโป · อย่าต่อยอดจาก
   สภาพที่จำไว้ แม้เพิ่งดูไปเมื่อเทิร์นที่แล้ว
2. **ห้ามแก้ไฟล์ที่อีกฝั่งมีงานค้างยังไม่ commit** — รอบนี้ `src/react/hooks/useTheme.ts`
   ถูกแก้ค้างไว้ (เป็นต้นเหตุเทสแดง 4 เคสด้วย) ผมเลือกรายงานแทนที่จะแก้ทับ
   แล้วบอก user ว่าทำไม · แก้ทับไฟล์ที่อีก agent กำลังเขียน = เละทั้งคู่
3. **commit เฉพาะไฟล์ของตัวเอง** (`git add <paths>` ไม่ใช่ `git add .`) เพราะ
   tree มีของคนอื่นปนอยู่
4. งานที่ปลอดภัยที่สุดในสภาพนี้คือ**สิ่งที่อีกฝั่งทำไม่ได้** — QA จริงในเบราว์เซอร์
   ด้วย session จริง (ดู [[reference_oc2plus_member_app_local_qa_session]]) และ
   การวินิจฉัยว่าเทสที่แดงอันไหนของจริงอันไหนเป็น env

เชื่อม [[feedback_parallel_sessions_git_safety]] [[reference_parallel_sessions_duplicate_symbols]]
