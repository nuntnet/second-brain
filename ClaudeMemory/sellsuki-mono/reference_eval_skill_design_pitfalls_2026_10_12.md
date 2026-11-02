---
name: reference-eval-skill-design-pitfalls-2026-10-12
description: "Lessons from three /eval-skill rounds on nunt-mode: pick a task with real design uncertainty, strip eval evidence and the profile from baseline worktrees, one run per cell is noise, subagents die at the org spend limit."
metadata:
  type: reference
---

1. **The task decides which tier you test.** A design settled by a house rule plus sibling code (SideBar gates) is tier 1 and every run, with or without the profile, goes straight to one design. For tier 2 pick a shape with no precedent and a fork you can settle by running something (the permission-orphan detector qualified).
2. **Worktrees cut from HEAD carry the experiment with them.** After the first commit of `nunt-mode.md` and its evidence folder, every new worktree contained the profile, its Eval section and the label map. Delete `.claude/agents/<mode>.md` and `docs/cards/evidence/skill-<name>/` in the baseline worktrees, and cut the Eval section out of the planted copy.
3. **Hide the label map from the judge** (use a private copy of the criteria without it) and shuffle labels. A `grep -v '^Map ('` failed silently because `grep` is aliased to rg here; use python to filter.
4. **One run per cell is noise.** Round 1 had opus+profile worst, round 2 had opus+profile best. Use 2 or more per cell before saying anything about the profile.
5. **Opus candidates take 50 to 65 minutes** on repo tasks; budget for it. Subagents die mid-work with HTTP 429 when the org monthly limit is hit (2026-11-01), leaving half-written worktrees. Check the dead ones with `git status` before reusing them.
6. A `head -20` on a grep produced a false "nothing calls it" in my own verification. Judge claims against the unfiltered search.

Related: [[project-nunt-mode-agent-profile]], [[feedback-head-on-grep-is-sampling-not-verification]], [[feedback-compare-command-outputs-with-rtk-proxy]].
