---
name: project-po-workflow-pack-not-on-main-2026-11-02
description: "2026-11-02: a new PO could not find /po-team because the sellsuki workflow lives only on fix/AI-49-safe-worker-activation. PO pack (67 files) opened as monorepo MR !52 into main, mergeable, NOT merged; engineering/pstack half still has no MR."
metadata:
  type: project
---

A new PO (น้อง PO) pulled the monorepo and `/po-team` was missing. Measured 2026-11-02: origin/main has 11 commands and 2 agents, AI-49 has 38 and 13; AI-49 is 1,939 commits ahead and main 71 ahead (diverged); no MR AI-49 → main. 59 files that mention pstack, nearly all absent on main (only `feature.md`, `land.md`, `repos.md`, `land.py`, `review-bot-lessons.md` exist there, in older form).

**Proposed first MR (branch cut from origin/main, e.g. `refactor/AI-49-po-workflow-pack`, never merge AI-49 whole):** 11 PO command/agent files, knowledge `cost-model, epic-index, roadmap-source, ux-ui-contract + schema`, skills `next-card, write-prd`, the whole `.claude/principles/` (38 files), `scripts/ux-ui-check.py`, `docs/sellsuki-roadmap*.md`, `generate-roadmap-md.js`, and `docs/control-tower/data.js` (the conflict: main 402 lines vs AI-49 640, control-tower already diverged, MR !29). Leave out `router.md`, verifier, trail-reviewer, inbox, ship-check, `nunt-mode` and the wave exception in `shipping.md`.

**Open:** nunt must choose PO pack first (recommended) or both halves at once; whether the 5 locally uncommitted po-* edits get committed first; po-team's eval status unchecked. The repo's `.mcp.json` carries only codegraph, so Jira for PO agents comes from each person's own Atlassian connector, merging cannot supply it.

**Quick workaround for the PO now:** checkout branch `fix/AI-49-safe-worker-activation`. Related: [[reference-monorepo-docs-live-on-ai49-branch]], [[project-team-workflow-sharing-goal]], [[reference-control-tower-data-js-diverged-main-vs-ai49]], [[project-pstack-as-workflow-benchmark]].

**Update 2026-11-02 12:20:** nunt said "ทำต่อเลย" and the PO pack went out as **monorepo MR !52** (`refactor/AI-49-po-workflow-pack`, aaef8822, 67 files, mergeable, no pipeline, Jira AI-49 comment 47409). Built from the committed state of AI-49 (the 5 locally uncommitted po-* edits are not in it). Dropped `/principles` and `rules/principles.md` (auto-loaded for everyone), `impeccable`, control-tower README. Dangling on purpose: `scripts/inbox.py`, control-tower README, user-level impeccable skill. `data.js` takes the AI-49 version (402 → 640 lines): merging !52 settles the control-tower question. Naive closure over file mentions ballooned to 174 files (README → verifier → doctor → codegraph); use one level of direct references plus judgment, not a fixed point.
**Still open:** nobody merged !52; the PO has not run `/po-team` end to end; no MR yet for the pstack/engineering half (router, principles rule, verifier, ship-check, inbox, nunt-mode), and `shipping.md`/`feature.md` on main are the old versions.
