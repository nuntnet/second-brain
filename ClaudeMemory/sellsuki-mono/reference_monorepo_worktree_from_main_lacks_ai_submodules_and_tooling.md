---
name: reference_monorepo_worktree_from_main_lacks_ai_submodules_and_tooling
description: "A monorepo worktree cut from `main` has NO chat-core/ai-agent/rag-core/messaging submodules, no router.md, no scripts/inbox.py and no AI ledgers (all live on fix/AI-49-safe-worker-activation); work the AI chat repos there by `git worktree add --detach` of each submodule inside it, and land ledgers onto AI-49 with read-tree/update-index/commit-tree"
metadata:
  node_type: memory
  type: reference
  originSessionId: a2d3f3d9-d7d8-471c-a770-28e0b1eef5b9
  modified: 2026-10-28T14:56:49.247Z
---

Seen 2026-10-28 on the AI-340/341 side session (worktree `mystifying-jemison-b19d85`, cut from `main`).

- `main`'s `.gitmodules` stops before the AI chat repos: `backend/sellsuki-chat-core`, `backend/sellsuki-ai-agent`, `backend/rag-core`,
  `backend/sellsuki-messaging-backend`, `frontend/ai-chat-admin-frontend` are only on `fix/AI-49-safe-worker-activation` (as are
  `.claude/rules/router.md`, `scripts/inbox.py`, `docs/cards/AI-*.md`, `docs/decisions/INBOX.md`).
- `git checkout -b <x> fix/AI-49…` inside the worktree aborts half-way (sandbox denies `.claude/skills`, `.claude/hooks` writes).

**How to apply:**
- Code: `git -C /Users/nunt/sellsuki_mono/backend/<repo> fetch origin main` then
  `git -C … worktree add --detach <worktree>/backend/<repo> origin/main`; branch there; remove the worktree before the turn ends.
- Ledgers / inbox: edit copies in the worktree (`git show fix/AI-49…:docs/decisions/INBOX.md > docs/decisions/INBOX.md`, run
  `INBOX_PATH=… python3 /Users/nunt/sellsuki_mono/scripts/inbox.py add …`), then commit onto AI-49 without checking it out:
  `GIT_INDEX_FILE=$TMPDIR/x git read-tree fix/AI-49…; git update-index --add --cacheinfo 100644,$(git hash-object -w f),f; write-tree; commit-tree -p fix/AI-49…; git branch docs/<card> <sha>`.
- Decision ids collide: the main checkout's INBOX.md carries uncommitted rows (D-046..048 on 10-28) and side branches carry more
  (D-049/050 on docs/AI-216…); check `grep -oE '^\| D-[0-9]+'` across all three before numbering.
- Related: [[reference_monorepo_docs_live_on_ai49_branch]], [[reference_auto_mode_blocks_glab_mr_create]],
  [[reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash]].
