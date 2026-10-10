---
name: feedback-generator-must-not-wipe-shared-dirs
description: "2026-10-11 my build-plugin.py did shutil.rmtree(plugins/sellsuki-workflow) and deleted another session's untracked plugin (scripts backing .githooks commit-msg/pre-push, hooks.json, five hand-written skills); recovered from a t3 checkpoint ref; rule — a generator writes only files it marks as its own and never wipes a directory in a tree shared by other sessions"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-10T18:28:59.423Z
---

What happened: I wrote `scripts/build-plugin.py` to derive `plugins/sellsuki-workflow/`
from `.claude/` and started it with `shutil.rmtree(OUT)`. That directory already held
another session's **untracked** work: `scripts/{card-new,commit-msg,lib,pre-push,
stop-check}.sh`, `hooks/hooks.json` and hand-written skills `card`, `ac-tests`,
`fanout`, `prove-red`, `sharpen`. `.githooks/commit-msg` and `pre-push` (core.hooksPath)
`exec` those scripts, so my next commit and push failed with "No such file". The
`[sellsuki-workflow] this branch is for AI-49` warnings I had been seeing all day came
from that very script. I noticed only because the hook broke.

Recovery: `git log --all -- 'plugins/*'` found the tree in `refs/t3/checkpoints/…`
(checkpoint `5e8b5291`) and in commits `03c84555` / `af5c486a` on a lineage that is not
an ancestor of the branch; `git archive <ref> plugins/sellsuki-workflow | tar -x` restored
it. Then I committed the restored files so the hooks no longer depend on untracked work.

**Why:** this workspace is one tree shared by several Claude sessions and Codex
(`repos.md`: "other sessions and Codex have uncommitted work in the same trees"). A
directory being empty in `git ls-tree HEAD` says nothing about whether it is empty on
disk. `rmtree` is `git add -A` in reverse.

**How to apply:**
- Before creating or regenerating any output directory, `ls` it and `git status
  --short` it; if anything is there that you did not write, stop and read it.
- A generator owns only files that carry its marker line; it skips everything else
  and never removes a directory. (`build-plugin.py` now does this.)
- A hook that `exec`s a repo path is a dependency: grep `.githooks/` and
  `core.hooksPath` before touching anything under `plugins/` or `scripts/`.
- If something is lost, look in `git log --all --oneline -- <path>` including
  checkpoint refs (`refs/t3/checkpoints/*`) before declaring it gone.
Related: [[feedback-parallel-sessions-git-safety]], [[feedback-fix-must-reach-everyone]].
