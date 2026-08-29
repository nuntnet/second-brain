---
name: project-monorepo-mainline-is-not-main
description: "The sellsuki_mono working mainline is chore/ai-mvp-local-run, not main — main is 2 months stale and diverged, not merely behind"
metadata: 
  node_type: memory
  type: project
  originSessionId: cac73755-22e9-4905-b12d-0973d98763b2
  modified: 2026-08-29T16:57:13.850Z
---

In `/Users/nunt/sellsuki_mono`, `main` is **not** where work happens. As of
2026-08-29 the active branch `chore/ai-mvp-local-run` is 432 commits ahead of
`main`, and `main` is 12 commits ahead of *it* — **diverged in both directions**,
so landing the branch on main is a merge with real conflict surface, never a
fast-forward. `main`'s last commit is 2026-06-26; the working branch commits daily.
There is no `origin/main` ref at all (see [[reference-monorepo-no-origin]]).

Several Claude sessions commit to this **same working tree on the same branch**
concurrently — HEAD moved under me twice in one session (7b9041e, a44603a). Also
~34 git worktrees under `.claude/worktrees/`, each on its own branch.

**Why:** anything you add to the monorepo root — `.claude/rules/*`, `scripts/*`,
`Makefile`, `.claude/settings.json` — only reaches people whose branch contains
it. Landing a shared convention on `chore/ai-mvp-local-run` does **not** reach a
branch cut from `main`, and a worktree on an old branch keeps the old rules file
indefinitely. That silently bit the codegraph fix
([[reference-codegraph-context-needs-projectpath]]).

**How to apply:** when a change is meant for everyone, say explicitly which
branches actually get it and which don't, rather than assuming "committed" means
"rolled out". Don't propose merging to `main` as a cleanup — it is a diverged
2-month gap and a product decision. Before committing, re-check HEAD: another
session may have committed since you last looked. Stage explicit paths, never
`-A` (see [[feedback-parallel-sessions-git-safety]]).
