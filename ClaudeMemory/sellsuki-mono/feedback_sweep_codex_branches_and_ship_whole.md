---
name: feedback_sweep_codex_branches_and_ship_whole
description: "When the PO says work must be usable now, sweep every AI-chat repo for Codex's unmerged branches and ship Codex's release branches whole (they are supersets of my feature branches) instead of re-carving; a Codex ledger that says BLOCKED on 'coordinator confirmation' or 'human publication' is not a merge blocker for the PO"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-11T02:40:51.937Z
---

On 2026-10-11 the PO asked "ยังมีงานใน branch อื่นๆแบบนี้อีกมั้ย กวาดส่งงานให้หมด ต้องรีบใช้แล้ว" after
finding the staging Sources tab lacked the Drive OAuth picker that worked on local.

**Why:** Codex works on long-lived `codex/AI-<card>-complete-*-release` branches across
chat-core, rag-core, ai-agent, admin-fe and knowledge-connector, pushes them without opening
MRs (branch pipelines are skipped; CI runs only in MRs), and its ledgers end in
"BLOCKED — request human confirmation that other sessions will not change the shared stack"
or "await independent human publication". Nobody answers those, so finished work sits for days
and the PO sees "not on staging" as the only signal. Those release branches already contained
every AI-298 commit of mine (`git cherry origin/codex/... <my-branch>` → all patch-equivalent or
superseded; `git diff --stat` showed the release as a strict superset), so my plan to carve
main-based sub-branches would have been slower and would have conflicted.

**How to apply:**
1. Sweep first: `for-each-ref` over local + `origin/*` branches ahead of `origin/main` in every
   AI-chat repo, then `glab api .../merge_requests?state=all` to separate merged from unmerged,
   then `git cherry`/`merge-base --is-ancestor` to see which small branches the release branch
   already holds. Report the sweep as a table before touching anything.
2. Ship the release branch whole, one MR per repo, in producer-first order
   (connector → rag-core → agent → chat-core → FE). Local `go build ./... && go vet` from the
   worktree Codex left at that head is enough pre-MR; CI decides the rest.
3. Treat Codex's "BLOCKED on human confirmation" as a note for the PO, not a reason to wait;
   the human review/publication step moves to staging after deploy.
4. Still read the branch's new env vars and CI gates before merging: on this sweep the FE
   branch added `scripts/check-google-drive-build.sh`, which fails the staging build unless
   `VITE_GOOGLE_PICKER_API_KEY` exists as a CI variable.

Related: [[feedback_user_runs_codex_in_parallel]], [[reference_auto_mode_blocks_glab_mr_create]],
[[feedback_list_open_mrs_before_opening_one]].
