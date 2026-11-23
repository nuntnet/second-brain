---
name: project_production_ccs_runs_hotfix_line_not_main
description: "Production CCS runs a v1.7.x hotfix line cut from tags (hotfix/v1.7.1 → v1.7.2, hotfix/v1.7.3 → v1.7.3/v1.7.4), ~200 commits behind main; fixes and env vars already on main (BOLA-311 company-name read, BOLA_API_BASE_URL/BOLA_SYSTEM_TOKEN) are NOT on production until back-ported"
metadata:
  type: project
---

Found 2026-11-20 when a production BOLA invite failed twice: (1) v1.7.2 500'd the join page on a missing company row
(main's invitationCompanyName, f9282855, was never released) → hotfix v1.7.3; (2) accept dialled `localhost:8080`
because v1.7.x values-production.yml lacks BOLA_API_BASE_URL/BOLA_SYSTEM_TOKEN (main has them) → v1.7.4 on
hotfix/v1.7.3 commit 26960db2, blocked on Vault token copy (CCS secret ≠ BOLA SYSTEM_ADMIN_TOKEN).

**Why:** main is 200+ commits past what production runs, so "fixed on main" says nothing about production CCS.

**How to apply:** for any CCS production bug, first `kubectl -n sellsuki get deploy` image tag, then check the fix with
`git merge-base --is-ancestor <fix> <tag>`. Back-port the minimum onto a `hotfix/` branch cut from the deployed tag;
diff main's values-production.yml against the tag's for missing env. Related: [[project_bola_invite_email_via_messaging_and_wave_2026_11_19]].
