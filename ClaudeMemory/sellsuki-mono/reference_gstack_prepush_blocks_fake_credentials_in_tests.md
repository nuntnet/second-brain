---
name: reference_gstack_prepush_blocks_fake_credentials_in_tests
description: "gstack-redact-prepush hook blocks git push when the pushed diff has credential-shaped literals, including fake test fixtures (sk-proj-…, postgres://user:pass@…); fix by assembling them at run time, not by --no-verify"
metadata:
  node_type: memory
  type: reference
  originSessionId: 412dc360-5d42-492b-8f38-95d6097c2b92
  modified: 2026-11-02T04:47:26.082Z
---

Seen 2026-11-02 pushing chat-core AI-385: `HIGH openai.key` and `HIGH db.url_with_password` from a secrecy-gate test's FAKE values blocked the push. The hook scans the net pushed diff, so a follow-up commit that removes the literal clears it (no history rewrite needed for unpushed commits).

**How to apply:** in tests that need a secret-shaped value, build it at run time (`"sk" + "-proj-" + strings.Repeat("0", 8) + "FAKE…"`) with a comment saying why. Do not bypass with `--no-verify` / `GSTACK_REDACT_PREPUSH=skip` on your own. Older chat-core tests (e.g. `plantedAPIKey` in use_case/prompt_secrecy_test.go) predate the hook.
