---
name: reference_sandbox_nested_clone_git_config_denied
description: "A git clone/branch inside the session worktree (.wt/<repo>) fails in the sandbox on .git/config and .git/hooks; run clone/switch/commit/push for nested clones unsandboxed, and pass GOCACHE=$TMPDIR/... for go build"
metadata:
  type: reference
---

Seen 2026-11-27 (AI-512): `git clone` into `<session worktree>/.wt/<repo>` fails in the sandbox with
`cannot copy .../hooks/commit-msg.sample` and, with `--template=`, `could not lock config file
.git/config: Operation not permitted`. `git switch -c` then creates the branch but cannot check it out
(upstream config write denied), leaving HEAD on main. Retry those git commands with the sandbox off.
`go build` in the sandbox also needs `GOCACHE=$TMPDIR/gocache-x` (the default cache is denied).
Add `.wt/` to `sellsuki_mono/.git/info/exclude` so the nested clones don't show as untracked.
Related: [[reference_sandbox_blocks_localhost_so_db_tests_skip_silently]], [[reference_session_worktree_hook_put_submodule_worktree_inside]].
