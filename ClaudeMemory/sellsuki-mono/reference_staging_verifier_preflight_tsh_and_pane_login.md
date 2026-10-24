---
name: reference_staging_verifier_preflight_tsh_and_pane_login
description: Before promising an independent-verifier pass on staging-th, check two access facts first — kubectl needs an interactive tsh login (profile may be production-only/expired) and the Browser pane login does not survive the pane being closed (2026-10-24)
metadata:
  type: reference
---

2026-10-23/24 BOLA wave verifier run produced zero verdicts twice, both times for access, not for code:

- **Night window (21:00–09:00):** staging-th API, accounts and kubectl all time out; the CloudFront SPA shell still answers 200, which looks like "up". Probe `bola-api.staging-th.sellsuki.com/health` and accounts, not the web host.
- **kubectl:** `tsh status` showed only the *production* profile, expired; the staging context's exec plugin then fails with "please use tsh login/tsh kube login manually". Renewal is an interactive SSO login, so only the user can do it. This blocks every log ACs and every "call from inside the pod" AC (the system token must not leave the cluster).
- **Browser pane session:** the user said they had logged the pane in, but the pane had been closed when the next turn began; reopening it landed on the Kratos login with no `bola_token`. The agent does not type credentials. Check `localStorage.bola_token` in tab `seed` before listing rows as drivable.

**How to apply:** at the start of a staging verification, run `/health`, `kubectl -n bola get deploy`, and read `bola_token` in the pane, then list the blocked rows with their prerequisite in one checklist instead of retrying. Also check what is actually on `main` per repo with `git merge-base --is-ancestor` before driving a card: BOLA-311's join app (`frontend/sellsuki-invitation`) and CCS backend halves are on feature branches only, so its J/W/P/R/F/M ACs cannot pass on staging as deployed.

Related: [[reference_sandbox_blocks_go_cli_tls_and_ssh]], [[reference_night_window_is_staging_th_only_production_runners_run]], [[reference_bola_staging_loki]].
