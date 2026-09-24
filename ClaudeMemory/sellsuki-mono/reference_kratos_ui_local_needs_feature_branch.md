---
name: kratos-ui-local-needs-feature-branch
description: Local kratos-ui-go cannot run plain origin/develop — signup breaks (no DISABLE_CONSENT) and develop's kratos.yml is a stale v1.3/127.0.0.1 config; merge has 7 conflicts incl. go.mod
metadata:
  type: reference
---

As of 2026-09-25 the local stack runs `backend/kratos-ui-go` on `feature/self-register-app-launcher` (16 commits not on develop, 73 behind). Switching to `origin/develop` looks harmless but:

- **Signup silently fails**: `DISABLE_CONSENT` (local `.env`) only exists on the feature branch. On develop, no consent option → `pdpa_reference_id` missing → admin CreateIdentity 400 → form just bounces back.
- **develop's `kratos_schema_client/kratos.yml` is stale** (v1.3.0, `127.0.0.1:4455` URLs, last touched 2026-03-17). The local Kratos container (v25.4) mounts this file; the working config is the feature branch's + local uncommitted edits (accounts.sellsuki.local URLs). Keep a copy before any checkout.
- develop has **deleted `.air.toml`**; feature branch has it.
- `git merge-tree` develop × feature = 7 conflicts (.gitignore, Makefile, go.mod/go.sum — `ory/client-go` vs `ory/kratos-client-go/v25`, env.go, css).

To read develop code without disturbing the stack: `git archive origin/develop | tar -x -C <scratchpad>`.
See [[local-kratos-mail-mailslurper]], [[kratos-ui-registration-bypasses-kratos-policy]].
