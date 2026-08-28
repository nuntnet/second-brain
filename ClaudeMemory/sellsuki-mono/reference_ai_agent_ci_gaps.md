---
name: ai-agent-ci-gaps
description: "sellsuki-ai-agent CI: code_analyse always fails (Sonar URL unset, allow_failure); gosec enforces G118"
metadata:
  type: reference
---

`backend/sellsuki-ai-agent` `.gitlab-ci.yml` jobs and their real behaviour:

- **`code_analyse_merge_request_amd` fails on every pipeline, repo-wide.** `CODE_ANALYSE_SERVER_URL` is not set, so sonar-scanner dies with `Failed to query server version: Expected URL scheme 'http' or 'https' but no scheme was found for /api/v...`. It is `allow_failure`, so the pipeline still reports **success**. Verified on MR !6 pipeline #56140 (job failed, pipeline success) and MR !7. **Do not chase this — it is not your change.** Someone should set the CI variable.
- **`sast_gosec` blocks.** It reports MEDIUM+ confidence only (a pre-existing G101 LOW in `src/entity/prompt_scan/scan.go:37` is present locally but not reported by CI). G118 ("goroutine uses context.Background while request-scoped context is available") **will fail the pipeline** — hit on AI-88.
- Jobs that block: `unit_test_merge_request_amd`, `integration_test`, `dependency_scan_govulncheck`, `sast_gosec`, `build_merge_request_amd`.

**Convention the review bot enforces:** `t.Parallel()` on every test — 20 existing test files in `src/` already do it, and there is a commit `b5d5ec2 test(context-assembler): mark case_checkpoint tests parallel (MR !4 finding 2)`. Exception that is legitimate and must be documented in-file: a test using `t.Setenv` cannot be parallel (Go panics).
