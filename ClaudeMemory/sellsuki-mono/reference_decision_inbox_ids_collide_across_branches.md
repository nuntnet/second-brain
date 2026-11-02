---
name: reference_decision_inbox_ids_collide_across_branches
description: "scripts/inbox.py assigns max+1 from the INBOX.md on the CURRENT branch, so parallel sessions mint the same D-ids; sweep every branch/worktree for the true max first"
metadata:
  node_type: memory
  type: reference
  originSessionId: 77b80035-3131-4ced-9fb5-6ef6750029a3
  modified: 2026-11-02T10:46:34.430Z
---

`inbox.py add` numbers an entry as max(ids in this branch's docs/decisions/INBOX.md)+1. Parallel sessions each work
on their own `claude/*` branch, so they hand out the same ids. On 2026-11-02 the coordinator said "highest is D-157,
use D-158+", while a sibling (AI-393) had already used D-171..D-173 and this branch's own file stopped at D-151, so a
plain `add` minted D-152 (already taken elsewhere).

**How to apply:** before adding, find the real max across every ref and worktree:
`git for-each-ref` over refs/heads+remotes `git show $b:docs/decisions/INBOX.md`, plus `cat .claude/worktrees/*/docs/decisions/INBOX.md`,
grep `^| D-[0-9]+`. Then rename the freshly added row's id (python replace on `| D-NNN | <date> | <card> |`). Leave a gap
above the known max when several sessions run at once. Tell siblings the range you took in the shared notes folder.
