---
name: feedback_keep_urgent_thread_visible
description: "When a big new topic opens mid-session (e.g. provider/white-label), keep the earlier urgent thread visible — user worried I'd forget the still-bleeding BOLA ownerless-workspace problem"
metadata:
  type: feedback
---

User, 2026-09-28, after a long provider/white-label detour: "ผมกลัวว่า คุณจะไป focus ปัญหาเรื่อง provider, white label จนลืมเรื่องก่อนหน้านั้นที่คุยค้างไว้ ซึ่งด่วนและสำคัญไม่แพ้กัน".

**Why:** the earlier thread (sweep creating ownerless BOLA workspaces every 5 minutes, 4 green MRs waiting, OC-4581 blocked on BOLA-331) was still causing damage while the new architectural topic had no ongoing damage. I had also linked the new card as *blocking* the urgent fix, which would have delayed it.

**How to apply:** when a new large topic starts, keep a short "still open" list and put anything with *ongoing* damage first. Before linking a new design card as a blocker, check it doesn't hold back an urgent stop-the-bleeding fix — split the urgent slice out. Related: [[feedback_no_tiny_cards_bundle_as_ac]], [[reference_bola_sweep_binds_without_an_owner]].
