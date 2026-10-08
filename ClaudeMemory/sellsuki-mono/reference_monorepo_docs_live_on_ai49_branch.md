---
name: reference_monorepo_docs_live_on_ai49_branch
description: monorepo root ใช้ branch fix/AI-49-safe-worker-activation เป็นสายหลักจริง — origin/main ค้างตั้งแต่ ส.ค. ไม่มี docs/release/environments.yaml · ledger การ์ด commit ลง AI-49 · local นำหน้า origin หลายสิบ commit ของ session อื่น อย่า push แทน
metadata:
  node_type: memory
  type: reference
  originSessionId: 53f9c3a5-efcb-4605-8af2-6e152327b40b
  modified: 2026-10-08T12:50:48.165Z
---

เช็กเมื่อ 2026-10-02 ใน `/Users/nunt/sellsuki_mono` (monorepo root ไม่ใช่ submodule)

**สถานะ branch**
- `origin/main` ค้างอยู่ที่ commit วันที่ 2026-08-27 และ**ไม่มีไฟล์** `docs/release/environments.yaml`
- งาน docs ทั้งหมดอยู่บน `fix/AI-49-safe-worker-activation` ซึ่งนำหน้า main ราว 1,091 commit ได้แก่ ledger ของการ์ด (OC-3559, OC-4599 ฯลฯ), environments.yaml และ `.claude/rules`
- local ของ branch นี้นำหน้า origin ราว 87 commit ส่วนใหญ่เป็นของ session อื่นที่ยังไม่ได้ push

**How to apply:**
- commit ledger หรือ environments.yaml ลง branch นี้โดยตรง อย่าแตก worktree จาก main
- stage เฉพาะ hunk ของตัวเอง เพราะ environments.yaml มักมีส่วนที่ session อื่นแก้ค้างไว้ (เช่นส่วน prod)
- ถ้าจะ push ต้องถามผู้ใช้ก่อน เพราะจะพางานของ session อื่นขึ้นไปด้วย

ดูประกอบ [[reference_control_tower_data_js_diverged_main_vs_ai49]]
