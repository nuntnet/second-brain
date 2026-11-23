---
name: feedback-review-other-agents-runbooks-against-git-rules
description: "Runbooks/plans written by Grokbot or other agents (e.g. ~/Downloads/monday-runbook.md, 2026-11-22) can prescribe rebase + force-push and stale \"known failure\" claims — review them against our rules and GitLab before executing"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-23T03:22:42.246Z
---

nunt runs other agents (Grokbot, Codex, other Claude sessions) that prepare deploy runbooks in parallel. The 2026-11-23 Monday runbook told the operator to `git rebase origin/main && git push --force-with-lease` (or press GitLab's Rebase button), called an ai-agent test "already red on main" when main's pipeline was green, and listed rag-core !30 as ready while the MR itself said "Do not merge yet". nunt said "แก้ runbook เลย" when shown these.

**Why:** our git rules forbid rebasing pushed branches and force-push; a runbook from another agent carries no authority over them, and its status claims are snapshots that drift.

**How to apply:** before executing a handed-over runbook, GET every MR's head SHA/pipeline/description from GitLab, translate rebase steps to `git merge --no-ff origin/main` + normal push, and treat "known failure" claims as unverified until main's pipeline confirms. ~/Downloads is not writable from the Bash sandbox — edit such files with the Edit tool. Related: [[feedback-git-flow-main-develop-main]], [[feedback-handoff-claims-check-ledger-and-mr]].
