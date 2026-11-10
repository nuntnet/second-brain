---
name: reference-commit-only-my-lines-of-a-shared-dirty-file-via-update-index
description: "When a file (docs/decisions/INBOX.md) already carries another session's uncommitted lines, commit only my rows by building the index blob from HEAD + my lines (git hash-object -w, git update-index --cacheinfo); git add -p is not available here"
metadata:
  node_type: memory
  type: reference
  originSessionId: 300c6962-12fa-4740-9aa3-3eb3ab997d45
  modified: 2026-11-10T16:06:14.548Z
---

`docs/decisions/INBOX.md` in the monorepo root routinely holds other sessions' uncommitted rows (2026-11-10: D-320 and D-328..D-338 were in the working tree, not in HEAD). `inbox.py add` appends mine to the same file, so `git add docs/decisions/INBOX.md` would commit their rows under my message, and interactive `git add -p` is not supported in this harness.

What worked:
1. `git show HEAD:<path> > head.md`, read the working file, pick out only my rows (regex on my `D-` ids).
2. Write `HEAD content + my rows` (inserted after the last `| D-` row) to a scratch file.
3. `blob=$(git hash-object -w scratch.md)` then `git update-index --cacheinfo 100644,$blob,<path>`.
4. Check `git diff --cached --stat` shows only my insertions and nothing else is staged, then `git commit` with no pathspec (a pathspec would re-read the working file and pull the other rows in).

Afterwards `git diff -- <path>` still shows the other sessions' rows as unstaged, which is the intended state. Related: [[feedback_parallel_sessions_git_safety]], [[reference_decision_inbox_ids_collide_across_branches]].
