---
name: project-ai-chatcore-merge-order
description: "chat-core AI stack merge order — fix/AI-32 must land in main BEFORE the 9-branch feature stack, else the 42P10 GetOrCreateOpen bug ships"
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-07T18:03:12.115Z
---

`backend/sellsuki-chat-core` carries a 9-deep unmerged feature stack (AI-33 → AI-34 → AI-47 → AI-56 → AI-20 → AI-35 → AI-54 → AI-55 → AI-58, plus `fix/chat-core-review-fixes` on top) built during the 2026-08 autonomous run — see [[project-ai-sprint234-autonomous-run]].

**Merge-order requirement (verified 2026-08-06 by review):** `fix/AI-32-getorcreateopen-generic-plan` is NOT an ancestor of that stack and is NOT on `main`. Branches below `feature/AI-48-conversation-assignment` therefore still contain the original 42P10 partial-index `ON CONFLICT` pattern in `src/repository/chat_session_repository/postgres_gorm.go` (`GetOrCreateOpen`, hot inbound path — see [[pg-partial-index-onconflict-generic-plan]]). **Merge `fix/AI-32` into `main` first, then merge the stack** — otherwise anything shipped before AI-48 carries the bug.

Update 2026-08-08: the AI-48 agent independently hit and fixed the same bug (its concurrent-claim test tripped it), so the fix now also lives inside `feature/AI-48-conversation-assignment` and everything stacked above it. Expect a trivial conflict when `fix/AI-32` merges — same intent, two authors.

Related known gap from the same review: `workspace.EnsureDefault` has a second partial-index arbiter that pre-exists on `main` and is NOT covered by the AI-32 fix — needs its own card.

Also: migration numbering in the stack runs 0004–0011 (0012 next), each registered once, no collisions — but they only apply cleanly in stack order.
