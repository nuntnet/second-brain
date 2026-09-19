---
name: helm-installs-cronjob-without-pulling
description: "A CronJob deploy job SUCCEEDS with a wrong image name — Helm installs it without ever pulling. The only evidence is ErrImagePull in a pod one schedule interval later. rag-core's CRON_IMAGE_NAME is the image repo, not SERVICE_NAME."
metadata:
  type: reference
---

Hit 2026-09-20 deploying rag-core's first CronJob.

## The failure that reports success

`deploy_cron_staging_arm` → **success**. Pipeline green, job log clean, nothing
in Helm's output wrong. Five minutes later the first tick:

```
Failed to pull image ".../service/rag-core-python:63c80ca3" ... not found
Error: ErrImagePull
```

Helm installs a CronJob **without pulling its image** — there is no pod at
install time. So the layer that reports the outcome is not the layer that can
fail. On a `*/5` schedule it then reproduces every five minutes, unwatched.

Same shape as [[reference_boot_guard_pins_an_old_image_silently]]: green
everywhere the deployer looks, broken where nobody does. **After deploying a
CronJob, wait one schedule interval and look at the pod.** A successful deploy
job is not evidence the CronJob works.

## The specific trap in rag-core

`CRON_IMAGE_NAME` is the image **repository**; `SERVICE_NAME` is the service.
In this repo they differ and every other name in `.gitlab-ci.yml` is the
service name, so the wrong one reads as obviously right:

| | value |
|---|---|
| `SERVICE_NAME`, Deployment, Service, pod prefix | `rag-core-python` |
| image actually pushed | `service/rag-core` |

They come from different sources — the cron template composes
`registry.fountain.sellsuki.com/service/${CRON_IMAGE_NAME}` itself, while the
API's deploy takes `--set image.repository=${REPOSITORY_URL}` from the SRE
template. In most repos those agree, which is why nobody writes it down.

**Get the truth from the build job's trace, never from the values file:**

```bash
glab api "projects/<url-encoded path>/jobs/<build job id>/trace" \
  | grep -oE "registry\.fountain\.sellsuki\.com/[a-z0-9/_.-]+:?[a-z0-9]*" | sort -u
```

Guarded now by `tests/unit/deployment/test_cron_image_name.py`, which asserts
the *wrong* value (CRON_IMAGE_NAME must not equal SERVICE_NAME) because the
right one has no in-repo source of truth.

Related: [[reference_pipeline_retry_runs_skipped_deploy_jobs]] ·
[[reference_ci_history_and_dns_are_not_deploy_status]]
