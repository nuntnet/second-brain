---
name: feedback-parallel-sessions-git-safety
description: "User runs multiple concurrent Claude sessions in the same monorepo — submodule working trees and the root index hold OTHER sessions' WIP; strict pre-mutation checks required"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-02T01:56:12.039Z
---

The user runs **several Claude sessions in parallel on the same monorepo checkout**. Observed collisions (2026-07-23): `backend/bola-backend` working tree was on another session's branch (`feat/bola-294-reply-token`) with uncommitted WIP; `frontend/sellsuki-invitation` had uncommitted F8 WIP that an agent folded into a commit (later had to evict `.env.dev` localhost values); the monorepo root index held the user's pre-staged items (`.gitmodules`, `backend/oc2plus-line-crm-service-3rdparty-api`) which twice got swept into my commits; an `--amend` at root once rewrote the OTHER session's ref-bump commit (recovered via reflog `reset --soft`).

**Why:** a failed `git checkout` does NOT stop subsequent file edits — my patch once landed on the other session's branch because the string also existed there. `git add -A` and `commit --amend` are the two biggest foot-guns in this environment.

**How to apply (hard rules for this workspace):**
1. Before ANY mutation in a submodule: `git branch --show-current` + `git status --porcelain` — verify the branch is MINE and note foreign dirty files.
2. If my branch isn't checked out, use `git worktree add` (scratchpad) instead of switching the shared checkout.
3. Never `git add -A` in a tree with foreign WIP; stage explicit paths and re-check `git status` before committing.
4. At monorepo root: stage ONLY intended submodule paths; never amend HEAD without confirming it's my own unpushed commit (`git log -1` author/message).
5. Fixing an accidental edit on a foreign branch: only restore files that were NOT in the foreign dirty list (`git checkout HEAD -- <file>`).

**Update 2026-08-02 — duplicate-WORK collision (not just git):** another session had already implemented+merged AI-12/AI-95 while this session was about to re-do them from a stale plan ("ทำไปแล้วอีก Session นะ"). Extended rules for parallel sessions sharing a Jira-driven backlog (see [[ai-chat-platform-plan]] for the live example):
6. Before claiming any card: check its Jira status (In Progress/In Review = taken) AND verify the target repo's remote — `git ls-remote --heads` + `glab mr list --all` — a stale conversation summary is NOT ground truth.
7. Claim before coding: transition the card to In Progress + drop a 🔒 comment naming the branch, so the other session sees it.
8. Pick disjoint lanes by repo: work in a fresh scratchpad clone, never in the shared monorepo submodule tree another session is sitting on.
