---
name: feedback_sprint_sweep_board_column_means_deploy_not_ac_proof
description: "2026-10-14 sprint sweep: 'Ready to test (staging)' = code on main of every repo + deployed; QA proves the ACs there. Don't hold a card in RTT(DEV) for unproven ACs. The real OC2Plus bottleneck is unmerged main twins, not PO decisions; merge-wave.py + the sweep recipe"
metadata:
  node_type: memory
  type: feedback
  originSessionId: e5b57a1c-03b4-4838-8275-26b6728d42a6
  modified: 2026-10-14T14:05:01.639Z
---

User asked (2026-10-14): "ถ้าไม่ติด PO แล้ว ดันทุกใบใน sprint ให้ success หรือ Ready to test (staging)". Four
read-only sweep agents applied the ledgers' stricter rule (every AC proved on dev before moving) and recommended
"keep" for almost everything — wrong lens for the board.

**Why:** the board columns mean *where the code is deployed for QA*. RTT(DEV) = on dev, RTT(staging) = on staging.
An AC with no evidence is what the column exists to collect; it goes into the Jira comment as "ให้ QA ตรวจ", not
into a reason to hold the card. "Done" is still never set by the agent (shipping.md §1).

**What the sweep found (reusable):** in OC2Plus the blocker is ~25 `main` twin MRs nobody merged (develop has the
code, main = staging does not, because of the OC-4736 divergence). All 11 red pipelines were runner failures
(`stuck_or_timeout_failure` / `runner_system_failure`), zero script failures. 6 of 8 "conflicts" were only the
generated `spec.gen.go` (fix: merge main, `make gen-http-fiber`, build/test, push). `has_conflicts` flips to true
the moment someone else merges to main — re-check right before merging. Some twins were never even pushed
(OC-4561) and some branches never got an MR (OC-4592, OC-3126 file-exporter).

**How to apply:**
1. Board → `sprint130.tsv` (JQL `sprint = "OC Sprint N" AND statusCategory != Done`, fields only; parse the saved
   file with python). MR map via `glab api projects/<enc>/merge_requests?state=…&updated_after=…` over the 13
   OC2Plus-adjacent projects, regex `OC-\d{4}` on title+branch+description — `glab mr list` has no `--state`.
2. Classify every open MR: merged / draft / conflict / green / runner-red / script-red (`scripts/merge-wave.py --plan`).
3. Night = conflict fixes + opening missing MRs (offline); morning = `scripts/merge-wave.py --deploy <wave>`
   (retry by job id, merge, play `deploy_staging_th_arm`), then move cards with a comment naming what QA must prove.
4. Jira writes one call per turn (parallel calls collide); `transitionJiraIssue` + comment must be two calls
   (the `update.comment` form demands ADF).
Evidence of the run: `docs/cards/evidence/sprint130-sweep-2026-10-14.md`.
