---
name: feedback-compare-command-outputs-with-rtk-proxy
description: "The rtk hook compresses command output (e.g. go vet to '1. … // ... 19 more lines'); when diffing two runs of the same command (develop vs branch) read both with `rtk proxy <cmd>` or the comparison is of a truncated list against a full one. 2026-10-11"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-11T14:26:16.959Z
---

On 2026-10-11 I compared `go vet ./...` on a develop baseline and on a merge branch. The branch run was written to a file through a pipe (full text), but the baseline went through the rtk output filter and came back as "1. src/…  // ... 19 more lines (total: 20)". The diff said "17 findings only on the branch" — an artifact. Re-reading the baseline with `rtk proxy go vet ./...` gave 17 vs 17, identical sets.

**Why:** rtk saves tokens by summarising output when it is printed, and the summary is not marked as lossy in a way a diff notices. A comparison needs both sides raw.

**How to apply:** before diffing outputs of the same command across two trees (vet, test lists, ship-check, grep results), run both through `rtk proxy` (as a command prefix, not inside a shell function) or redirect to files and compare the files. If one side shows "… N more lines" it was filtered. Related: [[feedback-head-on-grep-is-sampling-not-verification]], [[reference-tmpdir-differs-sandboxed-vs-unsandboxed]].
