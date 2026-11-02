---
name: project-nunt-mode-agent-profile
description: "2026-10-12 nunt-mode agent profile built with /my-mode: two layers (effort tiers 0-4 + 29 working-style rules mined from memory/ledgers/transcripts/git). First eval verdict REVISE (profile 11/7, none 8/9). Re-run on a tier 2 task before offering to the team."
metadata:
  type: project
---

`.claude/agents/nunt-mode.md` (root commit 3498ad62 on fix/AI-49-safe-worker-activation, not pushed). The user wanted a poteto-mode-like mode: poteto-mode is both style and rigor (design-space exploration, adversarial review, prove-on-real-artifact, autonomy), so the file has Part 1 (tiers + escalation triggers) and Part 2 (style). The 29-rule checklist was accepted without drops; rules 6 and 22 use the wording I proposed.

Eval (docs/cards/evidence/skill-nunt-mode/): one real symptom, 4 candidates, blinded judge. The opus run WITH the profile widened scope and rewrote an existing test's expected value, so tier 1 and rules 13 and 22 were tightened. One run per cell is noise-level.

**How to apply:** do not tell the team it is shared until a tier 2 re-eval passes and `make setup` is confirmed to install `.claude/agents/nunt-mode.md`. Related: [[project-pstack-as-workflow-benchmark]], [[project-team-workflow-sharing-goal]].

**Round 2 (2026-10-12, SideBar permission gates, 8 candidates):** profile mean 9.25 vs none 7.0 (sonnet 8 vs 7.5, opus 10.5 vs 6.5); the round-1 opus scope creep did not recur. But the judge ruled the task was NOT tier 2: house rule plus five unconditional sibling entries settled the design, and no run compared two designs by experiment. So the tier-2 machinery is still untested. Evidence: `docs/cards/evidence/skill-nunt-mode/tier2/`.

**Round 3 (tier 2 for real: a ship-check detector for permission codes referenced in code but seeded by no rps migration) is INCOMPLETE:** the org hit its monthly spend limit (HTTP 429) on 2026-11-01 and 3 of 4 candidates died mid-work; only the no-profile sonnet run (perm-seed-3) finished. Worktrees `/Users/nunt/sellsuki_mono/.worktrees/perm-seed-1..4` still exist. Do not score round 3 from one run; rerun all four once the limit resets, then update the file's `## Eval` section.
