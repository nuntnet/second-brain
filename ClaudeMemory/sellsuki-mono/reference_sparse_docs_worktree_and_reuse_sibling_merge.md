---
name: reference_sparse_docs_worktree_and_reuse_sibling_merge
description: "Ledgers on fix/AI-49 without the .claude/skills sandbox abort: `git worktree add --no-checkout -b docs/<x> <wt>/wt/docs fix/AI-49…` + sparse-checkout docs/cards; and a wave+main merge a sibling unit already made (pure 2-parent merge) can be reused as your base instead of re-resolving"
metadata:
  node_type: memory
  type: reference
  originSessionId: 254d2db4-5507-4b01-8b97-5d16bfe72477
  modified: 2026-11-04T18:00:12.074Z
---

Seen 2026-11-05 (V2-6 session).

- **Docs worktree:** `git -C /Users/nunt/sellsuki_mono worktree add --no-checkout -b docs/<card> <session-wt>/wt/docs fix/AI-49-safe-worker-activation`,
  then `git sparse-checkout set --no-cone '/docs/cards/*' '/docs/cards/waves/*'` and `git checkout`. Only ledgers are checked out,
  so the sandbox-protected `.claude/skills` / `.claude/hooks` paths never get written; commit normally (unsandboxed). Simpler than
  the read-tree/commit-tree recipe in [[reference_monorepo_worktree_from_main_lacks_ai_submodules_and_tooling]].
- **Reuse a sibling's merge:** when wave branch + `origin/main` conflict, look for `git log --merges` on sibling card branches for a
  commit whose parents are exactly the two heads (`%p`); compare two siblings' trees (`git diff A B`) — if only comments differ,
  `reset` your unpushed branch onto it. Saves re-resolving the same 4-file conflict per unit.
