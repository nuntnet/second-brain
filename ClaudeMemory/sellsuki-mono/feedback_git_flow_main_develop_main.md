---
name: feedback_git_flow_main_develop_main
description: Git flow ใหม่ (2026-10-02) — แตก feature จาก main · merge develop เพื่อเทสบน cluster · ผ่านแล้ว merge branch เดียวกันเข้า main · ห้าม merge develop เข้า main · ห้ามแตกจาก develop
metadata:
  type: feedback
---

**กฎ (user ยืนยัน 2026-10-02, "git flow ใหม่ของเรา"):**
1. แตก feature branch จาก `main` เท่านั้น (ไม่แตกจาก develop, ไม่ stack บน feature อื่น)
2. เทสแล้ว merge เข้า `develop` เพื่อเทสบน cluster จริง (dev)
3. ผ่านแล้ว merge **branch เดียวกัน** เข้า `main` → เทส automate full suite → deploy · ไม่ผ่านแก้จนผ่านหรือ revert · จบ loop แล้วแตกใหม่จาก main
4. ห้าม merge develop → main (ลากงานที่ยังไม่ validate ขึ้น staging ทั้งก้อน — เหตุการณ์ 2026-09-17)
5. repo ที่ไม่มี develop (testing/* ทั้งหมด, space-go, rag-core ฯลฯ) ข้ามข้อ 2: แตกจาก main, MR เข้า main
6. ชื่อ branch `feature|enhance|fix|refactor/<CARD>-<message>`, `release|hotfix/vX.Y.Z` (+`-staging` หลัง tag เพื่อเทส staging) · MR มี card / detail / POW(optional) · ไม่ squash · ลบ branch หลัง merge main

**Why:** branch ที่แตกจาก develop ลาก commit ที่ยังไม่ผ่านเทสไป main ด้วย (2026-10-02 วัดได้ 29–89 commit ต่อ branch ของงาน BOLA/OC-4523/OC-4514) จึง promote ทีละ feature ไม่ได้

**How to apply:** worktree ตัดจาก `origin/main`; ตอนเปิด MR เพื่อเทสบน dev ให้ target develop, เมื่อผ่านค่อยเปิด MR ของ branch เดิมเข้า main; `scripts/local-stack.sh` สร้างจาก main (override `LOCAL_STACK_BASE=develop`) · rules อยู่ `.claude/rules/repos.md` "Git flow" · แทนที่ [[feedback_oc2plus_merge_to_develop]] เฉพาะส่วน "ห้ามเข้า main ตลอด" และ "แตกจาก develop"; ยังเกี่ยวกับ [[feedback_no_develop_to_main_promotion_mrs]]
