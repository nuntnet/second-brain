---
name: ai-board-in-review-means-merged-unverified
description: "On the AI (AI chat platform) Jira board, \"In Review\" means merged to main but not yet verified in a real environment — not \"MR open, awaiting code review\""
metadata: 
  node_type: memory
  type: project
  originSessionId: 01c88554-c2ad-4379-b428-9806eeccfe92
  modified: 2026-08-27T17:56:43.402Z
---

Audited epic AI-9 (Admin Panel) on 2026-08-28: all 13 children sitting in **In Review** had their branches **already merged into `main`** and their MRs merged or closed. Only one MR was open in the whole repo (`!13`, AI-192).

So on this board `In Review` does **not** mean "waiting for someone to review the MR". It means: **code landed, acceptance not yet verified against a running environment.** AI-96's own last comment spells it out — *"AC ข้อ 1 ยัง verify ไม่ได้"* because the screen was unreachable, not because the code was unfinished.

Consequences when reading this board:

- Do **not** offer to "review the open MRs" for In Review cards — there usually are none.
- Do **not** treat In Review as work in flight that another session may be coding. Check `git branch -r --merged origin/main` first.
- The real blocker for an In Review card is almost always a **missing backend or missing environment**, so the useful next step is unblocking verification, not writing more frontend.
- `Done` is effectively unused on this board even for shipped work (AI-9 had 26 children and **zero** Done), so a To Do card may also be fully implemented — AI-184 and AI-149 both were. Never infer "not started" from To Do; grep the code.

AI-188 ("ปิด/แตก epic AI-3 กับ AI-9 ที่ยังเป็น To Do ทั้งที่ลูกเสร็จไปแล้ว") was closed to fix the *epic* statuses, but the children were never cleaned up — so the drift is still there.

**The one check that settles it** for the admin panel: open `apps/admin/src/main.tsx` in `frontend/ai-chat-admin-frontend` and look at the `unavailable` block near `bootstrap()`. Every port listed there is swapped for a `createUnavailable*Adapter()` outside mock mode, i.e. that screen has **no backend** and renders an honest error in production no matter how complete the UI looks. `packages/adapters/src/unavailableAdapters.ts` says so itself: *"the presence of an entry here is the todo list"*. A working mock and a working backend are indistinguishable from the screen — this file is the only place that tells them apart.

I got this wrong once (2026-08-28): moved AI-149 to In Review because its hook and banner were on `main`, when the card's own scope was "replace the mock with the real AI-155 read model" and the mock was all that existed. Read the card's scope table before judging from file existence.

Related: [[ai-board-stale-cards]], [[verify-as-the-user-sees-it]], [[jira-mcp-search-quirks]].
