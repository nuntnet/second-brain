---
name: reference-ccs-env-topology
description: "CCS backend deploy topology: dev env = ns sellsuki-dev ON the staging-th cluster (develop branch), ns sellsuki = staging (main); Loki there doesn't index these pods"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T08:30:21.637Z
---

`sellsuki-central-control-backend` environments both live on the **eks_staging_th cluster** (Teleport `tsh` kube access):

- ns **`sellsuki-dev`** ← `develop` branch, CI job `deploy_development_th_arm` (ARM nodes, `node-role/application-arm` toleration)
- ns **`sellsuki`** ← `main` branch (staging env)
- image tag = `CI_COMMIT_SHORT_SHA` of the branch head; the app logs it as `version` — fastest way to confirm which commit is running

**Why:** debugging a "failed develop deploy" by looking at ns `sellsuki` events is a red herring — that's main's env with its own history. Hit this 2026-08-11: !253's deploy timeout looked like a code crash but was a transient helm `--atomic --wait` timeout (ARM scheduling); plain job retry succeeded.

**How to apply:** for CCS deploy failures, check `kubectl -n sellsuki-dev` (develop) vs `-n sellsuki` (main) — match namespace to the failing job name first. Loki on this cluster (`system-logging` ns, labels `namespace_name`/`pod_name`) returned nothing for these pods — don't burn time there; retry the deploy and `kubectl logs` the crashing pod live instead.
