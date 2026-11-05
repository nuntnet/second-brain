---
name: subagents-share-scratchpad-helper-scripts-get-overwritten
description: "Sub-agents write into the parent's scratchpad; a helper script there (mutate.py) was silently replaced, so a prove-red run applied nothing and the test looked \"green under mutation\""
metadata:
  node_type: memory
  type: reference
  originSessionId: 6d416b86-7ae5-47ba-9ab1-ff959fe23b7c
  modified: 2026-11-05T08:54:22.164Z
---

On 2026-11-05 (chat-core coverage for !206) I kept `scratchpad/mutate.py` for prove-red runs, then spawned two sonnet agents and gave them the same scratchpad path. One wrote its own `mutate.py` over mine. My next spot-check ran the new script, which printed nothing, changed nothing and exited 0. The test then "passed under mutation", which looked like a weak test, but no mutation had been applied.

**How to apply:** give each sub-agent its own subfolder (`scratchpad/<agent>/`) and use uniquely named coordinator scripts (`mut_coord.py`). After every mutation, confirm it applied (`git diff --stat <file>` non-empty, or the script's own "applied N" line) before you read a green result as "the test did not catch it". Related: [[prove-red-script-never-pipe-into-head]].
