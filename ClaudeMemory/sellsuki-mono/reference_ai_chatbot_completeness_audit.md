---
name: reference-ai-chatbot-completeness-audit
description: The central AI-chatbot completeness audit at docs/audits/ai-chatbot-2026-09-07/ — where MVP scope, per-flow disposition and the AC ledger live, and what its "done" means
metadata:
  type: reference
---

`docs/audits/ai-chatbot-2026-09-07/` (snapshot 2026-09-07, read-only external
audit) is the authoritative baseline for "how far is the AI chatbot MVP".
Start here rather than re-deriving from the Jira board.

- `README.md` — verdict, scope boundaries, a 15-row **flow matrix** (F01–F15,
  R01, D01–D03) giving per-flow code/integration disposition and the exact
  evidence each flow still needs, ~11 source-pinned AC adjudications, and a
  prioritized workstream backlog.
- `jira-ledger.md` — all 213 cards with dispositions · `jira-requirements.json`
  — extracted AC sections for 183 cards · `acceptance-ledger.json` — 1,418
  mechanically split AC items, initially unadjudicated ·
  `acceptance-assessments.json` — the explicit targeted overlays ·
  `repo-evidence.json` / `supporting-repos.json` — commit pins and MR topology ·
  `onboarding-follow-up.md` — F01 decision-history reconciliation.

**Its definition of done is the important part:** an AC counts only with an
immutable code reference, the exact test command/job, the tested SHA, and the
real-vs-mock boundary. "MR merged" is explicitly not completion, Jira workflow
state is tracked separately, and unknown may never be counted as passed — so no
completion percentage is defensible. It also forbids a "not started" verdict
from an absent AI key or a To Do status without checking branch
patch-equivalence first (see [[reference-ai-board-stale-cards]],
[[reference-ai-board-in-review-means-merged-unverified]]).

Consequence for planning: as of the snapshot **0 flows are staging-verified**
and R01 (build → deploy → smoke → rollback) has no proven deployed tuple, so
code that lands before the release path works becomes inventory that cannot be
verified. Scope baseline is platform plan §6.1 plus the later case/playbook
additions and the self-service Facebook pilot requirement; platform-track cards
are NOT automatically pilot blockers.
