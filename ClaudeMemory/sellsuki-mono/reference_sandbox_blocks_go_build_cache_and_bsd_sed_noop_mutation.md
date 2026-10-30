---
name: sandbox-go-cache-and-bsd-sed-mutation
description: go vet/test fail in sandbox on the go-build cache; and a prove-red mutation via macOS sed -i silently did nothing
metadata:
  node_type: memory
  type: reference
  originSessionId: 50e8ca89-54bf-4490-8ea8-d5db607c8c11
  modified: 2026-10-30T10:29:40.024Z
---

- `go vet ./...` / `go test` in the sandbox fail with "open ~/Library/Caches/go-build/...: operation not permitted"; run them with the sandbox disabled (the build alone printed success, so vet is the one that shows it).
- A worktree for a submodule branch can be made under the session scratchpad (needs unsandboxed `git worktree add`); full chat-core suite `go test ./src/... ./cmd/... -p 1` takes >10 min, run it in the background.
- Prove-red trap: `sed -i 's/…/…/' file` on macOS (BSD sed) errors "unescaped newline" without `-i ''`, leaves the file untouched, and the test then passes. That looks like "mutation did not go red". Mutate with a python exact-match replace (assert count==1) and confirm `git diff --stat` shows the change before running the test. See [[feedback_prove_red_script_never_pipe_into_head]].
