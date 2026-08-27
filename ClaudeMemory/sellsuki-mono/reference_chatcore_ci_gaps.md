---
name: reference-chatcore-ci-gaps
description: "chat-core: integration_test never runs on main (MR-only rule), and shared-template MR jobs are pinned to staging-th so they starve when that pool is down"
metadata:
  node_type: memory
  type: reference
---

Two structural CI gaps in `backend/sellsuki-chat-core`, both hit on 2026-08-25.

**1. `integration_test` is `rules: $CI_PIPELINE_SOURCE == "merge_request_event"`.** The repository tests — the only ones that run against real Postgres/Redis — **never run on a push to `main`**. A bug can merge green and then fail somebody else's unrelated MR, which is exactly how the `ApplyPlaybookVersion` 23505 leak was found, and how a `src/guard` failure sat on main unnoticed.

**2. Shared-template MR jobs are pinned to one runner pool.** `unit_test_merge_request_amd` / `build_` / `code_analyse_` extend `.staging_th_runner_amd_tags`. GitLab assigns a job only to a runner whose tags are a **superset** of the job's, so when the staging-th pool went offline those jobs sat `pending` for 84 minutes and were eventually marked `failed` with `failure_reason: stuck_or_timeout_failure`, `started_at: None`. The MR reads "merge blocked: ci_still_running", which looks like progress.

**How to apply:**
- The env-agnostic anchor is **`.runner_amd_tags`** (`docker, general, sellsuki, amd`) in `templates/default.template.yml` of `sellsuki/sre/deployment/pipeline-deployment`. Both the Production and staging-th pools satisfy it. Override `tags:` only — redeclaring `extends` drops the rules/variables/script the template supplies. Done in chat-core MR !26.
- Safe for MR **test** jobs only; leave deploy jobs environment-pinned. Evidence it works: chat-core's own `integration_test`, `sast_gosec`, `dependency_scan_govulncheck`, `openapi_breaking_change` already use `tags: [docker]` and were the only jobs that kept passing through the outage — including `integration_test`, which needs live Postgres and Redis services.
- Validate any `.gitlab-ci.yml` edit with the project's `/ci/lint` API before pushing rather than guessing at syntax.
- A job failing with `system failure: prepare environment ... container not found ("helper")` is infra, not code — retry it. `sast_gosec` is a HARD gate here (not `allow_failure`), so an infra death looks exactly like a security finding; always open the log before concluding.

Complements [[reference_dead_staging_runner_tag]], which covers the older `staging` (no `-th`) tag having no runner at all.

---

**3. `code_analyse_merge_request_amd` fails on EVERY ref, including `main` — it is not your MR.** Confirmed 2026-08-27 across MR !22/!24/!25/!26/!27/!28/!29 and four `main` pipelines: always red, always in 0–18s. The trace shows sonar-scanner dying before it reads a single file:

```
ERROR Failed to query server version:
Expected URL scheme 'http' or 'https' but no scheme was found for /api/v...
```

`CODE_ANALYSE_SERVER_URL` (and likely `CODE_ANALYSE_TOKEN`/`PROJECT_KEY`) is unset as a project CI/CD variable, so the scanner builds a host-less URL. Setting those is a maintainer/SRE action, not a code fix. Treat a red `code_analyse` as background noise; never spend time bisecting it.

**4. `sast_gosec` has no timeout margin, so runner contention reads as a security failure.** Observed durations on comparable refs: 204s, 224s, 232s, 245s, 340s, 417s, 439s, 556s, 846s, 879s, 1106s — a **5× spread on the same branch and even the same commit** — against a fixed `1h0m0s` job timeout. On 2026-08-27 job 236150 (MR !28, commit `6025a718`) ran out the full hour and died with `execution took longer than 1h0m0s`; **retrying that identical commit passed in 275s** (job 236228) — a 13× swing with zero code change.

**How to apply:** when gosec fails, read the tail of the trace FIRST. `execution took longer than…` is contention, not a finding — just retry:

```bash
glab api --method POST "projects/:id/jobs/<id>/retry"
```

Do not go looking for what the branch "made slower": package footprint barely moves the needle (18 changed packages vs 11 on a job that finished in 880s). Same lesson as the `prepare environment` infra death above — gosec is a hard gate, so every infra failure impersonates a vulnerability.
