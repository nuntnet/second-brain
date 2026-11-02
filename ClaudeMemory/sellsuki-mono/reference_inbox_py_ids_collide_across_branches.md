---
name: reference_inbox_py_ids_collide_across_branches
description: "scripts/inbox.py add allocates max+1 from the branch's own INBOX.md, so parallel branches hand out the same D-id; coordinators assign a starting id (e.g. \"start at D-180\")"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8ed73ff7-86b0-452e-b6b7-bd58e788b8bb
  modified: 2026-11-02T08:55:44.921Z
---

`scripts/inbox.py add` numbers a new entry as max(D-ids in this branch's `docs/decisions/INBOX.md`) + 1. Parallel sessions on different monorepo branches each see a different file, so they mint the same id (2026-11-02: the local max was D-151 while D-152…D-173 already existed on sibling branches).

When a coordinator says "start at D-180", keep the script as the writer but pin the id: exec the part of `scripts/inbox.py` above `if __name__` with `__file__` set, call its `load()`, append the row with the chosen id and the same columns `add` writes, then `save()`. Then `inbox.py check <CARD>` reads it normally.

Related: [[project_workflow_tiers_and_inbox_2026_10_13]]
