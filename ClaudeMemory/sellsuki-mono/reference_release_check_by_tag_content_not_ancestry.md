---
name: reference_release_check_by_tag_content_not_ancestry
description: "Is card X on prod?" — compare the prod tag's CONTENT with main; CCS/rps/console tags hold cherry-picks, so merge-base --is-ancestor gives false "unreleased"
metadata:
  type: reference
---

Checked 2026-11-28 during the AI board production verifier round. CCS v1.8.5, rps v1.3.1 and console v1.2.7 contain card commits under **different shas** (cherry-picked onto a release/hotfix line). `git merge-base --is-ancestor <sha> <tag>` says "not released" while the code is identical.

**How to apply:** use `git diff <tag> origin/main -- <files the card touched>` (empty = released), or grep the tag for the change. Only call a card "waits on release" after that. Example of a real one: chat-core v1.3.6 did not contain AI-509's `slogAuditor` `.Write()` fix (`32f1e920`).

Related: [[project_production_ccs_runs_hotfix_line_not_main]] (marked outdated, but the cherry-pick effect is real).
