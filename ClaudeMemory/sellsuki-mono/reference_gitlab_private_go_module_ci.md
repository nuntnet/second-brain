---
name: reference-gitlab-private-go-module-ci
description: Cross-repo private Go module fetch in GitLab CI needs job-token allowlist + git credential rewrite in .gitlab-ci.yml — recurring pattern for every service consuming ai-platform-kit-go
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8b54895a-0013-46d6-8588-dd94f45b9c4a
  modified: 2026-08-04T02:29:24.882Z
---

When a Go service's `go.mod` requires a private same-instance module (e.g. `gitlab.sellsuki.com/sellsuki/share/library/ai-platform-kit-go`), a plain `go mod download` in GitLab CI fails with:

```
invalid version: git ls-remote -q ...: fatal: could not read Username for 'https://gitlab.sellsuki.com': terminal prompts disabled
```

Two separate things are required to fix this, not just one:

1. **GitLab CI/CD job-token inbound allowlist** on the *target* (dependency) repo — Settings → CI/CD → Token Access → allowlist must include the *consuming* repo's project ID. Check/set via API:
   - `glab api projects/<target_id>/job_token_scope` (check `inbound_enabled`)
   - `glab api projects/<target_id>/job_token_scope/allowlist` (list)
   - `glab api -X POST projects/<target_id>/job_token_scope/allowlist -f target_project_id=<consumer_id>` (add)
   - Confirmed IDs: `ai-platform-kit-go` = 848, `sellsuki-chat-core` = 849 (added 849 to 848's allowlist 2026-08-01/02), `backend/entity` = 536, `sellsuki-messaging-backend` = 568.
   - ⚠️ 2026-08-02: the allowlist POST is **blocked by the permission classifier** for Claude (security-setting change) — the user must run it themselves. Pending as of 2026-08-04: add 568 to BOTH 536 and 848 allowlists (messaging-backend MR !10/!11 pipelines stay red until then; also note the 404 signature differs — with a valid netrc but missing allowlist, GitLab returns 404 on `?go-get=1`, not a username prompt).
   - GOPRIVATE also needed when the repo sets `GOPROXY: https://go.sellsuki.com,direct` — the proxy doesn't serve these modules, and without `go env -w GOPRIVATE=gitlab.sellsuki.com/*` the direct fallback misbehaves on nested subgroups.

2. **Git credential rewrite in `.gitlab-ci.yml`**, in every `before_script` that runs `go mod download` (the shared `pipeline-deployment` template's `UNIT_TEST_BEFORE_SCRIPT` / `INTEGRATION_TEST_BEFORE_SCRIPT` / `E2E_TEST_BEFORE_SCRIPT` variables, plus any job-local `before_script`):
   ```yaml
   git config --global url."https://gitlab-ci-token:${CI_JOB_TOKEN}@gitlab.sellsuki.com/".insteadOf "https://gitlab.sellsuki.com/" && go mod download
   ```
   Define once as a YAML anchor (`.go_private_module_auth: &go_private_module_auth >- ...`) placed before `variables:`, then reference with `*go_private_module_auth` everywhere `go mod download` was used.

**Why this matters going forward**: per [[project_ai_chatsystem_e0_structure]], every future AI-chatsystem service (AI-2/AI-3/AI-4/AI-5/AI-11, per AI-12's dependency list) will consume `ai-platform-kit-go` and hit this exact same two-part gap on its first CI run. Don't re-diagnose from scratch — apply both fixes together.

Validate `.gitlab-ci.yml` changes with `glab api -X POST "projects/<id>/ci/lint" -H "Content-Type: application/json" --input <(echo '{"content": "..."}')` before pushing.
