---
name: reference-shared-identity-infra-singleton
description: "Kratos/Hydra/Keto/kratos-courier run as singleton (1-replica) pods, all colocated on one node in share/share-dev namespaces on the staging-th cluster — one bad node takes down login for that whole env"
metadata:
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-14T03:07:54.416Z
---

Shared identity infra — `kratos`, `hydra`, `keto`, `kratos-courier`, `kratos-ui-go` — lives in namespaces **`share`** (staging/main) and **`share-dev`** (dev) on the `staging-th` EKS cluster (same cluster referenced in [[reference_ccs_env_topology]]).

**The actual bug class**: `kratos`, `keto`, and `kratos-courier` in `share-dev` are each single-replica Deployments/StatefulSet with **no anti-affinity**, so Kubernetes happily schedules all three onto the same node. When that one node has kubelet trouble (`NodeNotReady`), the entire dev-th login stack (`accounts.dev-th.sellsuki.com/self-service/login/browser`) goes down together — not a Postgres/DNS problem, not app code, just a scheduling gap.

**Confirmed recurrence**: hit twice in one session on 2026-08-13/14, both traced to node `ip-10-21-1-19.ap-southeast-7.compute.internal` flapping (kubelet stopped posting status; pods showed matching restart counts ~14h apart). A downstream symptom of the same instability window: `dpa/status` calls on `api.crm.dev-th.oc2.plus` returned 504 twice then 403 — plausibly Keto timing out mid-check then returning inconsistent results during recovery, though this wasn't confirmed via k8s (Teleport session expired mid-investigation).

**How to apply**: when `accounts.dev-th.sellsuki.com` or any OC2Plus dev-th login/permission-check starts erroring, check node health first (`kubectl get nodes`, look for `NotReady`) and whether kratos/keto/kratos-courier share a node, before assuming an app-level bug. The real fix (≥2 replicas + pod anti-affinity for these three) is SRE/platform scope — flagged, not something to fix via app code or read-only kubectl access.

**Access note**: kubectl here goes through Teleport (`tsh`); only `production` and `staging-th` contexts are available, read-only (get/describe/logs) approved. The `tsh` session expires and needs interactive SSO re-login — Claude cannot refresh it itself; ask the user when it dies mid-investigation.
