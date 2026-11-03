---
name: reference_sandbox_bind_eperm_fails_go_tests_and_slog_panics_in_tests
description: "In the sandbox, Go tests that start a local listener FAIL (not skip) with \"listen tcp 127.0.0.1:0: bind: operation not permitted\" (kit authctx, CCS repository httptest pkgs); and sellsuki-go-logger slog.Warn panics in unit tests where no logger is initialised"
metadata:
  node_type: memory
  type: reference
  originSessionId: f746364c-7c1d-4189-93bc-f067e13be78d
  modified: 2026-11-03T08:51:55.656Z
---

Two traps seen 2026-11-03 while fixing wave review findings:

1. **Sandbox bind EPERM looks like a real red.** `ai-platform-kit-go/authctx` (gRPC fake on
   127.0.0.1:0) printed ~10 `--- FAIL` lines; CCS `consent_document_repository`,
   `identity_repository`, `messaging_otp_config_repository`, `workspace_provisioning_repository`
   (httptest) likewise. Read the error text before believing it: `bind: operation not permitted` →
   rerun that package with the sandbox off; all passed. Different from
   [[reference_sandbox_blocks_localhost_so_db_tests_skip_silently]] (those skip, these fail).

2. **`slog.Warn(...)` from `github.com/Sellsuki/sellsuki-go-logger/v2` panics (nil deref, slog.go:97)
   in unit tests** of chat-core use cases — the global logger is only set up in main. Adding a log
   line to a path a test exercises turns the test into a panic. Record on the span
   (`sp.RecordError` with a wrapped cause) instead, or keep the log on a path no unit test hits.
