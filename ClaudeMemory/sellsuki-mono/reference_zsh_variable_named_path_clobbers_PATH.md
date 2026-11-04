---
name: reference-zsh-variable-named-path-clobbers-path
description: "In zsh a shell variable named `path` is tied to $PATH — `path=/x` or `for path in ...` wipes PATH and every later command is \"command not found\" (git, python3, curl); results then look like real failures"
metadata:
  node_type: memory
  type: reference
  originSessionId: f746364c-7c1d-4189-93bc-f067e13be78d
  modified: 2026-11-04T01:35:47.031Z
---

zsh ties the lowercase array `path` to `PATH`. Assigning it (`path=$(...)`, `for path in /health /readyz`) replaces PATH, so every following command in that Bash call fails with "command not found".

Hit twice on 2026-11-03/04: a health-probe loop (`for path in ...; curl`) and a project-path variable in a round-1 MR check, whose output then read "CONFLICT" for every repo — a fake result, not a merge conflict.

**How to apply:** never name a variable `path` (use `proj`, `u`, `route`, `p`); in subagent briefs that will run shell, say so. If a run shows `command not found: git`/`python3` mid-loop, suspect this first and discard the run's results. Related: [[reference_zsh_colon_modifier_eats_git_refspec]].
