---
name: project_bola_v2_cleanup_goal_2026_10_12
description: "2026-10-12 PO goal: clear all old BOLA cards (done→correct status, unneeded→postpone, needed→pull up) and write cards accurately BEFORE implementing, ahead of starting BOLA v2; board cleanup applied, rewrite phase pending"
metadata:
  node_type: memory
  type: project
  originSessionId: 37718d4a-c6c6-4da2-ad71-349c1d7ff383
  modified: 2026-10-12T17:09:45.891Z
---

User goal (2026-10-12): push all old BOLA work to finish before expanding to **BOLA v2**; "เน้นเขียนการ์ดให้ตรงก่อน จะได้เปิด implement ภายหลัง" — card quality first, implementation later.

Done that day: audited all 263 open BOLA cards against code/MRs, closed 66 (superseded/dup/placeholder), fixed ~60 statuses, re-sprinted into S8 (must-finish) / S9 (QA sweep) / S10 (next). Full plan + per-card evidence: `docs/bola-board-cleanup-2026-10-12.md` (verdict files were only in scratchpad).

**Why:** the board had two overdue active sprints, work from Aug–Oct in no sprint, and ~45 cards in Ready for QA that no one signs. Merged-to-main cards stay Ready for QA, never Done, because main = staging.

**How to apply:** next phase is /po-review + rewrite of S8/S10 cards (311 split by release slice, 187 vs 188 AC, 260, 205, 225, 273, 338/339; 227/228/229 contradict each other). Open decision for the PO: BOLA-334 (central vs per-provider BOLA) gates v2 topology. Sprints 1–2 still need a human to complete.
