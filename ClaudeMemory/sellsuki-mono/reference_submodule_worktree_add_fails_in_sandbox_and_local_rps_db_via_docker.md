---
name: reference_submodule_worktree_add_fails_in_sandbox_and_local_rps_db_via_docker
description: "git worktree add for CCS/rps fails inside the Bash sandbox (cannot write .claude/ .idea/ in the new tree) and leaves a branch with no worktree; local rps DB is reachable as postgres:postgres via the docker postgres, not the tool's envDefault"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ae058e-5385-445d-8aed-f8888c50db6a
  modified: 2026-10-28T16:55:48.619Z
---

Seen 2026-10-28 (AI-289):

- `git -C backend/<repo> worktree add …` **inside the sandbox** fails for any repo that tracks `.claude/` or `.idea/`
  (CCS, rps): "unable to create file .claude/agents/…: Operation not permitted", then "Could not reset index".
  The branch is created but the worktree is not registered and the directory is gone.
  Fix: `git worktree prune`, then `git worktree add <path> <existing-branch>` with the sandbox disabled.
  chat-core (no `.claude/`) works sandboxed.
- Go commands need the sandbox off too (build cache under ~/Library/Caches), and **TMPDIR differs** between the two
  modes, so a backup copied to `$TMPDIR` sandboxed is not there unsandboxed ([[reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash]]).
  For red proofs, commit first and undo the break with `git checkout -- <file>`.
- Local rps DB for read-only tool runs: `DATABASE_ROLE_POSTGRESQL_URI='postgres://postgres:postgres@localhost:5432/sellsuki_role_permission?sslmode=disable'`
  (docker-compose.yml creds; the tool's envDefault user `sellsuki` fails auth). No psql on the host: use
  `docker exec sellsuki_mono-postgres-1 psql -U postgres -d sellsuki_role_permission`.
- zsh: `set -- $x` does not word-split; write explicit calls in loops.
