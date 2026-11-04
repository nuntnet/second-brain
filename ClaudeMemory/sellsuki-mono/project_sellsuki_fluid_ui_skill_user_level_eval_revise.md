---
name: project_sellsuki_fluid_ui_skill_user_level_eval_revise
description: "sellsuki-fluid-ui skill is installed at USER level only (~/.claude/skills, from ~/Downloads/skill.zip, 2026-11-04); /eval-skill verdict REVISE — do not move it into the repo's .claude/skills yet"
metadata:
  node_type: memory
  type: project
  originSessionId: f746364c-7c1d-4189-93bc-f067e13be78d
  modified: 2026-11-04T01:35:47.185Z
---

nunt installed `sellsuki-fluid-ui` (UX-first, CSS-first motion, a11y, TH/EN) into `~/.claude/skills/` on 2026-11-04 after Claude read all 21 files (safe: text + a read-only script; `agents/openai.yaml` is Codex metadata). Every session and subagent now sees it.

`/eval-skill` the same day: REVISE. With the skill, sonnet runs framed the user need better (Need 3/3 vs 2/2) but still shipped repo-level defects the repo's own checks would catch (brand text 2.3:1 on white; a shared i18n key changed under another caller). Evidence and the one change to make: `docs/cards/evidence/skill-sellsuki-fluid-ui/README.md` (add "run the route's a11y/contrast e2e and grep callers of any edited key/component" to its verification step; rerun with isolated clones, opus cell lost to contamination).

**Why:** the evidence lives in the repo but nothing points a future session at it before someone proposes sharing the skill with the team.
**How to apply:** before suggesting the team adopt it or copying it into `.claude/skills`, apply the revise item and rerun the eval. Related: [[reference_eval_skill_design_pitfalls_2026_10_12]].
