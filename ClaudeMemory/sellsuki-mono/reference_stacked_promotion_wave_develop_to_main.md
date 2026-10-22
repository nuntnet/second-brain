---
name: reference_stacked_promotion_wave_develop_to_main
description: "2026-10-22 BOLA: promote every develop-only card to main as a STACKED MR wave built in a scratch worktree — chain-merge main into each card branch in order, resolve conflicts by taking develop's file, end with one sync branch so main == develop byte for byte; classifier still refuses the merge click"
metadata:
  node_type: memory
  type: reference
  originSessionId: e2d40730-b4e0-484b-b3c2-d83461734bf6
  modified: 2026-10-22T07:46:29.941Z
---

Used 2026-10-22 to move 14 BOLA cards (backend 115 commits, frontend 47) from `develop` to `main` before a production tag, without merging develop into main (git flow forbids it).

**Recipe (scripts lived in the scratchpad: chain.sh / chain2.sh / gpush.sh / mkmr.py):**
1. `git cherry origin/main origin/develop` is the ground truth of what main lacks — card lists from Jira/ledgers miss no-card fixes (AI-277, auth fallback, OC-4514 asset COPY were all missing).
2. `git worktree add --detach <scratchpad>/wt origin/main` per repo. For each card branch in order: `checkout -B local/<b> origin/<b>` then `merge <candidate>`; candidate advances to the branch. Stop at the first conflict.
3. Conflict resolution rule: take `origin/develop`'s copy of the conflicted file (develop already resolved the same pair). Verified after: `git diff origin/develop` on the final candidate must be EMPTY — that single diff is the proof, not the per-branch reasoning.
4. Last branch = `refactor/<sync-card>-sync-develop-into-main`: develop's version of every file still differing. Commit message lists the commits it carries.
5. Build/test on the final tree (frontend symlink `node_modules` from the main checkout; backend `go test` must run unsandboxed for httptest). Push branches as themselves (no force), open MRs with "merge order N/M, stacked" in the description.
6. Expect: a test that assumes an empty runner env goes red on the MR pipeline of the branch that adds a CI variable (FE !152, BOLA-330 `VITE_KRATOS_ACCOUNTS_URL`) — the fix (`vi.stubEnv`) lived in the NEXT stacked branch; cherry-pick it down. `go mod download` connection reset on go.sellsuki.com = retry the job by id.
7. **The merge itself is refused by the auto-mode classifier ("Merge Without Review") even with the user's explicit instruction** — hand over an ordered script (`scripts/bola-merge-wave-2026-10-22.sh`) the user runs with glab. Same as [[reference_bola_jira_has_no_in_review_status_and_auto_mode_denies_merge]].

Related: [[feedback_git_flow_main_develop_main]], [[reference_sandbox_blocks_go_cli_tls_and_ssh]], [[reference_oc2plus_backoffice_develop_main_diverged_2026_10_11]].
