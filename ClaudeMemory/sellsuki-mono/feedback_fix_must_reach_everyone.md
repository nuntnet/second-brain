---
name: feedback-fix-must-reach-everyone
description: "User expects \"fixed\" to mean it reaches every consumer — other sessions, worktrees, teammates, other branches — and to be told plainly which ones it doesn't"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: cac73755-22e9-4905-b12d-0973d98763b2
  modified: 2026-08-29T16:57:31.153Z
---

After I fixed codegraph and reported it working, the user asked two follow-ups
in a row: **"ไม่กระทบงานที่ session อื่นๆทำอยู่ใช่มั้ย"** and **"เรารันให้ทุกคนใช้งาน
code graph ได้จริงๆเลยมั้ย"**. Both were the right question and neither was
answered by what I had done. The fix worked for a new session in the main tree
and for nobody else: `make setup` never mentioned it, git worktrees silently
answered from the wrong branch, and the whole thing sat on a branch other people
don't build from.

**Why:** in this workspace a change lands in a tree shared by several concurrent
Claude sessions, ~34 worktrees on their own branches, and teammates on fresh
clones. "It works for me now" and "it works" are different claims, and the user
checks the difference. Auto-loaded rules are the sharpest case: a running session
holds the old file in context and a file edit cannot reach it.

**How to apply:** before reporting a shared-tooling change as done, enumerate the
consumers and check each — running sessions, worktrees, fresh clone via `make
setup`, other branches, sub-agents — and state which are covered and which are
not, instead of a blanket "fixed". When touching shared state, verify explicitly
that nothing else was disturbed: don't kill other sessions' processes, stage
explicit paths, and confirm afterwards (`ps`, `git show --stat`, per-submodule
`git status`) rather than asserting it. Prefer a mechanism nobody has to remember
(a hook, a `make setup` step) over a documented command. See
[[project-monorepo-mainline-is-not-main]],
[[feedback-parallel-sessions-git-safety]], [[feedback-verify-as-the-user-sees-it]].
