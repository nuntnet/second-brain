---
name: reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash
description: "$TMPDIR in a sandboxed Bash command and in a dangerouslyDisableSandbox command (kubectl/tsh) point to different directories, so a file written by one cannot be found by the other. Use the session scratchpad's absolute path instead"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2b0f396b-5941-4a79-8e15-d9e0597e4520
  modified: 2026-10-27T11:30:06.675Z
---

Seen on 2026-10-27. A kubectl command (which must run unsandboxed because of TLS/tsh) wrote `$TMPDIR/rps10m.log`. The next command ran sandboxed with python and failed with `FileNotFoundError`. In the sandbox `TMPDIR=/tmp/claude-501`, while the unsandboxed run used a different directory.

**How to apply:** when work alternates between unsandboxed commands (kubectl, tsh, grpcurl into the cluster) and sandboxed ones (python/grep that read the result), write files to the scratchpad's absolute path (`/private/tmp/claude-501/<project>/<session>/scratchpad`). Never use `$TMPDIR`. Related: [[reference_sandbox_blocks_go_cli_tls_and_ssh]].
