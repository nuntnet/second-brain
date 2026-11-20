---
name: reference_unsandboxed_tmpdir_shared_log_names_collide
description: With the sandbox off, $TMPDIR is /var/folders/... shared by every session; a fixed name like unit.log reads another session's log. Use the scratchpad + a card-unique name.
metadata:
  type: reference
---

With `dangerouslyDisableSandbox` (needed for `go` here, see [[index_go_db_sandbox]]), `$TMPDIR` resolves to `/var/folders/hz/.../T/`, NOT the sandboxed `/tmp/claude-501/...` path. So:

- a file written to `$TMPDIR` in a sandboxed call is not there in an unsandboxed call (and vice versa);
- that folder is shared by every session on the machine: on 2026-11-20 (AI-502) `$TMPDIR/unit.log` held another session's frontend test output, and a grep on it reported "0 ok / no FAIL" for a Go run that had actually failed.

**How to apply:** write logs to the session scratchpad with an absolute path and a card-unique name (`<scratchpad>/ai502f-unit.log`), never a bare `$TMPDIR/unit.log`. If a log's content does not look like the command you ran, distrust it before trusting its counts.
