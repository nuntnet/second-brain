---
name: reference_oc2plus_backoffice_fe_red_on_develop
description: oc2plus-linecrm-frontend-backoffice ไม่เขียวบน develop — 618 type error + 136 test fail ต้องจด baseline ก่อนแก้
metadata:
  type: reference
---

**`frontend/oc2plus-linecrm-frontend-backoffice` ไม่เขียวบน `origin/develop`** (วัด 2026-09-12
ที่ `2f991f8`, worktree สะอาด ยังไม่แตะโค้ด):

| | baseline |
| --- | --- |
| `npx vue-tsc --build --force` | **618 error** ใน 34 ไฟล์ |
| `npx vitest run` | **136 fail** / 514 pass / 10 skip · 9 spec file fail จาก 85 |

ตัวอย่าง error เดิม: `base_rounding: string` ไม่ assign เข้า `BaseRounding`,
`MemberFilter` vs `CampaignFilter`, `RouteRecordName | null` ใน `router/index.ts`
· spec ที่ fail เดิม: `AppSwitcher.spec.ts`, `services/bola/mock.spec.ts` ฯลฯ

**How to apply:** ก่อนแก้อะไรในรีโปนี้ ให้**จด baseline ก่อน** แล้วเทียบทีหลัง ไม่งั้นจะ
(ก) นึกว่าตัวเองทำพัง หรือ (ข) อ้างว่า "เขียว" ไม่ได้เลย

```bash
npx vue-tsc --build --force 2>&1 | grep -cE "error TS"
npx vitest run 2>&1 | grep -E "^\s*FAIL" | sort -u > /tmp/baseline-fail.txt
```

เกณฑ์ที่ใช้ได้จริง = **"ไม่เพิ่มจาก baseline"** ไม่ใช่ "0" · ไฟล์ที่เราแตะต้องสะอาดของมันเอง
(`gofmt -l` / eslint 0 error) · เช็กด้วยว่า error ที่โผล่ในไฟล์ของเราเป็นของเดิมที่แค่
**เลขบรรทัดเลื่อน** เพราะเราแทรกโค้ด — เทียบกับ `git show origin/develop:<file>`

ดู [[reference_oc2plus_ci_outage_2026_09]] · [[reference_turbo_cache_crosssession_false_green]]
