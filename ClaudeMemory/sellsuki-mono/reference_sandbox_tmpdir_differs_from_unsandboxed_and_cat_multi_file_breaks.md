---
name: reference_sandbox_tmpdir_differs_from_unsandboxed_and_cat_multi_file_breaks
description: "$TMPDIR in a sandboxed Bash call (/tmp/claude-501) differs from an unsandboxed one (/var/folders/...); and `cat a b` is rewritten by the rtk hook into a broken `read` call"
metadata:
  node_type: memory
  type: reference
  originSessionId: aed3e5ad-eb79-4a65-b3e9-318e17615787
  modified: 2026-11-05T09:52:18.071Z
---

Seen 2026-11-05 (chat-core CI speed session):

- A file written to `$TMPDIR` by a `dangerouslyDisableSandbox` call lands in `/var/folders/.../T/`; a later sandboxed call's `$TMPDIR` is `/tmp/claude-501`, so the file "does not exist". Use the session scratchpad's absolute path for anything shared between sandboxed and unsandboxed calls.
- `cat file1 file2` through the rtk hook failed with `read: 'file1': not a valid identifier`. Use the Read tool (or `sed -n 1,200p`) instead. Related: [[feedback_rtk_cat_strips_comments_never_use_cat_to_rewrite_files]], [[reference_sandbox_blocks_go_build_cache_and_bsd_sed_noop_mutation]].
