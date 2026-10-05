---
name: project-team-workflow-sharing-goal
description: User wants to teach the OC2Plus team their fast AI-driven way of working and automate/share it; 2026-10-05 baseline numbers and the blocker found
metadata:
  node_type: memory
  type: project
  originSessionId: 20232f32-5776-48b9-9e35-4cb92ae88bee
  modified: 2026-10-05T05:24:54.129Z
---

Goal (2026-10-05): the user ships far more than the rest of the OC2Plus team and wants to teach that way of work and make their workflow automated and shareable.

Baseline measured 2026-10-05 (OC2Plus repos, origin/develop, since 2026-07-01, no merges): user 770 commits / 129 cards referenced / 78% commits carry a card id / ledger `docs/cards` only theirs; Soup 418 / 14 / 22%; Time 107 / 6 / 64%; Palm 58 / 2 / 19%. Test-file share and active days were similar, so speed is not from fewer tests or more days. 43% of the user's commits are outside 09:00-21:00.

Blocker found: the workflow (`.claude/` rules, `/feature`, `/land`, ship-check) lives only in the monorepo root, where the user is the sole committer; teammates work inside submodules. Of the OC2Plus repos only member-api tracks `.claude` files; backoffice-api, frontend-backoffice, frontend-member have none. Root workflow files were also uncommitted and `.claude/worktrees/*` was littered.

**Why:** the user asked how to coach the team, so re-measure against this baseline rather than re-deriving it.

**How to apply:** proposed order was commit the stray files, ship a shared plugin, enforce via pre-push `make ship-check` and a commit-msg hook for card ids, auto-create the ledger from Jira, then pair on one real card. Offered to mine past sessions for repeated prompts to turn into skills (prove-red, fanout, ac-tests, self-review, close-loop); user had not yet answered. Prefer measuring cards per week in working hours over commit counts.
