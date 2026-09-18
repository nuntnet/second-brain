---
name: reference_bola_dev_env_exists_and_is_live
description: "BOLA dev DOES exist — ns bola-dev on staging-th, auto-deploys from develop, and values-development.yml is live config; the repo's .gitlab-ci.yml has no development block because CI_JOB_ENABLE is set as a GitLab project variable that no grep can see"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-18T07:20:33.969Z
---

**Verified 2026-09-18 with `kubectl` against the real cluster**, after concluding
the opposite twice from reading files.

| fact | evidence |
|---|---|
| ns `bola-dev` exists | `kubectl get ns` — Active 111d |
| backend runs there | `back-office-of-line-api-backend` 1/1, plus 4 cronjobs |
| it auto-deploys from `develop` | image tag `5b54b459` = HEAD of `origin/develop`; rollout 13 Sep 15:47, merge was 15:40 |
| `values-development.yml` is LIVE config | `KAFKA_SERVERS=kafka-cluster-kafka-bootstrap.datastore:9092` on the pod, byte-identical to the file |
| chart | Helm, `helm-generic-deployment-0.1.0`, release `back-office-of-line-api-backend` in `bola-dev` |
| mode | self-host: `AUTH_MODE` is **not set** at all → `local_jwt` |

## 🔴 Why reading the repo gives the wrong answer

`backend/bola-backend/.gitlab-ci.yml` defines only `.variables_export_production`
and `.variables_export_staging_arm`. The word `development` appears **zero
times**. Following the SRE templates:

```
deploy_development_th_arm
  extends .variables_export_development_th_base   → CI_JOB_ENABLE: "false"
                                                    HELM_VALUES_PATH: deployment/values-development.yml
  extends .rules_check_development_th             → runs only if
                                                    CI_COMMIT_BRANCH == "develop" && CI_JOB_ENABLE == "true"
```

Every file says the dev deploy can never run. **It runs.** `CI_JOB_ENABLE` is set
to `true` as a **GitLab project-level CI/CD variable**, configured in the UI —
invisible to `git grep`, invisible to reading any template.

**The lesson, which cost three wrong conclusions in one session:** for anything
CI-configurable from a web UI, the repo is not the source of truth. Check the
running thing — `kubectl get deploy -o json` and compare an env var you know the
file sets. One command settles what an hour of template-reading cannot.

Wrong conclusions this produced, all stated confidently before being checked:
1. "BOLA has no dev environment" — it has one, 111 days old.
2. "`values-development.yml` is a dead file" — it is live config.
3. "BOLA-319 was closed Done with its dev half landing in a dead file" — it
   landed; the card was honest.

## Addresses on dev (for wiring anything to it)

- Kratos: `kratos-admin.share-dev` (port **80**, never `:4434`)
- CCS + rps: ns `sellsuki-dev`
- frontend: `https://bola-web.dev-th.bearyweb.com`, API `bola-api.dev-th.bearyweb.com`
- secret `bola-backoffice-secret` in `bola-dev` is missing exactly one key vs
  staging: `ORY_KRATOS_PUBLIC_URL`
- **no Oathkeeper rule exists for any bola dev host** (91 rules cluster-wide, 41
  of them dev, none bola)

Flipping dev to SaaS mode is tracked as **BOLA-330** (repo config) blocked by
**BOLA-329** (SRE). See [[project_bola_auth_mode_deployment]],
[[project_bola_deploy_topology]], [[project_bola_saas_kratos_deploy_gap]].
