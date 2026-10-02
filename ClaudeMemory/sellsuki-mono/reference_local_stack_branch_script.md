---
name: reference_local_stack_branch_script
description: scripts/local-stack.sh — branch local/stack ต่อ submodule (local-only, สร้างจาก origin/main) ไว้ merge feature ที่อยากดูรวมกันบน stack เดียวของเครื่อง; add/drop/rebuild/restore/status
metadata:
  type: reference
---

สร้าง 2026-10-02 หลังเจอว่าโฟลเดอร์หลักของ submodule ค้างบน branch ของ session อื่นหลายวัน (bola-frontend ค้าง `feat/PAT-2735-profile-menu` 3 วัน) และไม่มีที่ไหนบอกว่าใครถือ · `scripts/local-stack.sh add <repo> <branch>` จำ branch เดิมไว้ (`.git/local-stack/prev`), ปฏิเสธถ้า working tree มีงานค้าง, ไม่ตั้ง upstream (push-all.sh ข้าม `local/*`), conflict = ไม่ merge · `restore <repo>` กลับ branch เดิม · `LOCAL_STACK_BASE=develop` ถ้าอยากเห็นบนสถานะ dev cluster · selftest `scripts/local-stack-selftest.sh` (14 เคส) · กฎอยู่ `.claude/rules/repos.md` "Local stack"

**สถานะ:** ใช้ครั้งเดียว ตอนนี้ restore คืนโฟลเดอร์หลักแล้ว (local/stack ว่าง) · stack รันได้ต้องสตาร์ท `bola-api` ผ่าน overmind เอง (ผมห้ามสตาร์ทตรง) · ดู [[feedback_git_flow_main_develop_main]]
