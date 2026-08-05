---
name: reference-rtk-git-output-filtering
description: "rtk (Rust Token Killer) hook transparently rewrites git commands and can return compressed/empty output (e.g. \"ok ✓\", blank commit body) instead of real git data — use the full git binary path to bypass it when fidelity matters"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8b54895a-0013-46d6-8588-dd94f45b9c4a
  modified: 2026-08-05T15:59:39.083Z
---

Per `~/.claude/RTK.md`, a Claude Code hook transparently rewrites plain `git ...` commands to `rtk git ...` for token savings. This is usually fine, but its filtered/summarized output can be misleading when you need exact data:

- `git status --short` returned literally `ok ✓` instead of the real file list.
- `git log -1 --format="%H%n%s%n%n%b"` returned only the hash, with subject/body silently empty.

**Symptom to watch for**: git output that looks suspiciously compressed, or a field that should never be empty (commit subject/body) coming back blank.

**Fix**: call the real binary directly to skip the hook's rewrite/filtering, e.g. `/opt/homebrew/bin/git log ...` (find the path with `which -a git`). `rtk proxy <cmd>` (per RTK.md) is the documented alternative for raw output.

Use the direct-binary form whenever inspecting commit messages, diffs, or branch state that will inform a decision (e.g. verifying what a branch actually contains before merging) — don't rely on the hook-filtered form for anything beyond a quick sanity check.

**Also affects `glab` (2026-08-05)**: hook-wrapped `glab ci list` / `glab ci get` / `glab api` returned **stale cached pipeline status** — showed a pipeline as `success`/`failed` while the GitLab API said `running`/`waiting_for_resource`, and `glab ci get` kept serving job states frozen at an old timestamp. This produced a false "pipeline failed" and a false "pipeline succeeded" in the same session. Fix: `rtk proxy glab api "projects/:id/pipelines/<id>"` for authoritative status when deciding to merge/cancel; treat hook-filtered glab output as unreliable for CI state.
