---
name: reference_sre_gitops_repos_and_sync_policy
description: Where the dev/staging gateway + auth config lives in Git (api-gateway, bridge, ory-helm) and which ArgoCD apps auto-sync — merging is NOT enough for the manual ones; BOLA has no Application of its own
metadata:
  type: reference
---

Verified 2026-09-30. Opening/merging these MRs works with normal glab access (clone `git@gitlab.sellsuki.com:...`).

| what | repo · path | ArgoCD app | auto-sync |
|---|---|---|---|
| Emissary Host + Mapping | `sellsuki/sre/configuration/api-gateway` · `development-th/` / `staging-th/` (`hostnames-*.yaml`, `mapping.*.yaml`, apiVersion v3alpha1) | `api-gateway-development` / `-staging` | **OFF — click Sync after merge** |
| Oathkeeper Rule | `sellsuki/sre/configuration/bridge` · `manifest/<env>/rule.yaml` | `bridge-development` / `bridge-staging` | **OFF — click Sync** |
| Kratos config | `sellsuki/share/ory-helm` · `development-th/kratos-values.yaml` (multi-source helm) | `ory-krato-development` / `ory-kratos-staging` | ON — rolls out itself; shared by every dev product |
| octoplus/sellsuki/patona rules | own repo per product | `<product>-<env>-manifest` (`prune+selfHeal`, own AppProject) | ON |

- **BOLA has no Application/AppProject/repo of its own.** Its staging Rule `bola-api-auth` sits in `bridge-staging`, which is Bandscanner's app (SpiceDB + rules in ns `bandscanner`). Dev Rule (bridge !8) was put there too as a stopgap. Agreed direction: own `sre/configuration/bola` repo + AppProject + `bola-development`/`bola-staging` Applications like `octoplus-*-manifest`; dev first, staging migration needs care (unproven that Argo won't prune a resource whose tracking-id changes).
- Kratos allowlist: staging lists only `allowed_return_urls` for bola (no CORS entry) — CORS is not needed.
- Checking the live cluster needs `tsh kube login teleport.internal.staging-th.sellsuki.com`; a dead session or a restarting kube agent looks like "empty output", not an error — probe first.
- Rule: dev-th Kratos cookie `sellsuki_session_dev`, domain `dev-th.sellsuki.com` → an API on bearyweb never gets it (login loop, SRE-1684); BOLA dev API host is `bola-api.dev-th.sellsuki.com`.

Related: [[reference_oathkeeper_global_cookie_session_is_staging]], [[project_bola_auth_mode_deployment]].
