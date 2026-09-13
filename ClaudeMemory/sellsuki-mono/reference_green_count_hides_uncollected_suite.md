---
name: reference_green_count_hides_uncollected_suite
description: "ไฟล์เทสที่ collect ไม่ผ่านรายงาน 0 failure ทำให้ยอด \"failed\" เท่าเดิมทั้งที่เทสหยุดรันไปทั้งไฟล์ — ต้องเทียบยอด \"ผ่าน\" กับ baseline ไม่ใช่ดูยอด \"แดง\""
metadata: 
  node_type: memory
  type: reference
  originSessionId: d0b39379-feaa-4fd5-b115-18617c202159
  modified: 2026-09-13T15:44:08.738Z
---

**ยอดเทสแดงเท่าเดิม ไม่ได้แปลว่าไม่มีอะไรพัง** (เจอ 2026-09-13 ตอน implement OC-4523 บน `frontend/oc2plus-linecrm-frontend-member`)

การเพิ่ม import หนึ่งบรรทัดเข้า `AppShell.tsx` ทำให้ลำดับ module evaluation เปลี่ยน แล้ว `AppShell.spec.tsx` **collect ไม่ผ่านทั้งไฟล์**

```
Error: [vitest] There was an error when mocking a module...
Caused by: ReferenceError: Cannot access '__vi_import_7__' before initialization
```

สาเหตุ: `vi.mock('@/services/session', () => ({ ... new ErrUnauthorized(...) }))` ไป**อ้าง import ระดับบนของไฟล์** ซึ่ง vitest เตือนไว้ในข้อความ error เองว่าห้ามทำ เพราะ `vi.mock` ถูก hoist ขึ้นเหนือ import ทั้งหมด ของเดิมรอดมาเพราะลำดับ evaluation บังเอิญ init binding นั้นก่อน = โชค ไม่ใช่การรับประกัน

**สิ่งที่ทำให้มองไม่เห็น:** ไฟล์ที่ collect ไม่ได้ให้ผล **0 failure** (ไม่ใช่ N failure) ⇒ สรุปผลยังขึ้น `3 failed` เท่าเดิม ทั้งที่เทส 5 ตัวหยุดรันไปแล้ว agent รายงานว่า "baseline 3 failed, หลังแก้ 3 failed เหมือนเดิม" ซึ่งจริงตามตัวเลขแต่พลาดสาระ

**วิธีจับ:** เทียบ **ยอดที่ผ่าน** และ **จำนวนไฟล์** กับ baseline ไม่ใช่ยอดที่แดง
- baseline `origin/develop`: `Test Files 1 passed`, `Tests 5 passed` (รันเฉพาะไฟล์นั้น)
- branch เรา: `Test Files 1 failed`, `Tests no tests`
- ยอดรวมทั้ง suite: 1096 → 1152 ทั้งที่เพิ่มเทสใหม่ ~61 ตัว ⇒ หายไป 5

วิธีที่เชื่อได้คือแตก worktree `--detach` ที่ `origin/develop` แล้วรันไฟล์เดียวกันเทียบ

**แก้:** ให้ factory import เอง `vi.mock('@/x', async () => { const { Y } = await import('@/y'); return {...} })` assertion เดิมไม่ต้องแตะ

**ญาติกัน อีกเคสในวันเดียวกัน:** เทสความปลอดภัยที่ mock ให้ `getIDToken` โยน error แล้วยืนยันว่า token ไม่ถูก log — token ไม่เคยมีอยู่จริง เทสจึง fail ไม่ได้ พิสูจน์ด้วยการเติม `console.warn(token)` บนเส้นสำเร็จแล้วยังเขียว ⇒ **เทสความปลอดภัยต้องรันบนเส้นที่ของจริงมีอยู่ และควรมี control assertion ยืนยันว่ามันมีอยู่จริงตอนตรวจ**

**How to apply:** หลังรับงานจาก sub-agent อย่าเชื่อ "จำนวนแดงเท่าเดิม" ให้ diff ยอดผ่าน/ยอดไฟล์กับ baseline เสมอ และยิง mutation ใส่ assertion สำคัญเพื่อดูว่าเทสแดงจริง เกี่ยวกับ [[feedback_verify_absence_claims]] [[reference_test_stub_more_permissive_than_service]] [[project_oc4523_line_login_cards]]
