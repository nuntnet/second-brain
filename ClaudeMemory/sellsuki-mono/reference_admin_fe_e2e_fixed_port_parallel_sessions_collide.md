---
name: reference_admin_fe_e2e_fixed_port_parallel_sessions_collide
description: ai-chat-admin-frontend playwright.config.ts hard-codes port 5196 with reuseExistingServer:false — parallel card sessions (and a verifier) collide; copy the config to an untracked temp file with its own port; runs need unsandboxed Bash (port bind) and unsandboxed TMPDIR differs
metadata:
  node_type: memory
  type: reference
  originSessionId: b5b359bf-6b73-4b00-807e-134a240abc51
  modified: 2026-11-02T20:53:16.741Z
---

Seen 2026-11-03 (AI-372): my e2e run failed "localhost:5297 is already used" because the verifier subagent was serving on the
same temp port. The repo config pins `PORT: "5196"` (vite reads `process.env.PORT`), so two worktrees running e2e at once
always collide.

**How to apply:** `sed 's/5196/<port>/g' playwright.config.ts > playwright.<card>.config.ts` (untracked, delete after), give a
verifier a different port from your own, run `playwright test -c that.config.ts` with the sandbox off. Write logs to the
scratchpad absolute path ([[reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash]]). `-t`/`-g` take a regex: a `+` in a test
name matches nothing and the run "passes" with zero tests — the same trap as [[feedback_prove_red_script_never_pipe_into_head]].
