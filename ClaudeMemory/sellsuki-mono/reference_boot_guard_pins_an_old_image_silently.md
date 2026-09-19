---
name: boot-guard-pins-an-old-image-silently
description: "A boot-time config guard + Helm --atomic silently pins a service to an old image: cluster looks healthy, every new deploy rolls back, and the running pod predates the guard. rag-core sat 16 days behind this way."
metadata:
  type: reference
---

Found 2026-09-20 on rag-core (`rag-core-python` in ns `sellsuki`).

## The shape

1. Someone adds a guard to config validation — "APP_ENV is staging/prod, so
   env var X is required" — and refuses to boot without it.
2. X exists in no environment. Nobody notices, because the pod *currently*
   running was built **before** the guard and does not contain it.
3. Every subsequent deploy crash-loops. Helm `--atomic --wait` rolls back.
4. The cluster reports `1/1 Running`. `kubectl get deploy` is green. CI is
   green. Nothing anywhere says "this service is 16 days stale".

rag-core's `startup_errors()` demanded `GOOGLE_SHEETS_API_TOKEN`; the deployed
image stayed at `8a87c874` for 16 days while three whole routers' worth of
feature work sat in `main`, and the team reasoned for weeks about *wiring* a
KB feature whose endpoints were not in the cluster at all.

## How to check, in one command

Do not infer from CI history or from `kubectl get deploy` — both lie here.
Ask the running process what it serves:

```bash
kubectl port-forward -n sellsuki svc/<svc> 18001:80 &
curl -s localhost:18001/openapi.json | python3 -c \
  "import json,sys;[print(p) for p in sorted(json.load(sys.stdin)['paths'])]"
```

Compare against what the repo's router assembly mounts today
(rag-core: `src/interface/http_server/server.py`). A missing *router* — not a
missing field — means the image predates the feature entirely.

Also worth reading: `kubectl get rs -n <ns>` for ReplicaSets with 0 ready
replicas. A failed rollout leaves its ReplicaSet behind, and building a debug
pod from its spec reproduces the exact boot error the rollback erased.

## The design rule this teaches

A guard that protects an OPTIONAL capability must fail that capability, not the
process. rag-core is mostly retrieval and KB; neither touches Google, and
refusing to boot took both down for a feature nobody used. The fix was a
refusing adapter (`UnconfiguredSheetFetchRepository`) plus a non-retryable 503
(`ErrFeatureNotConfigured`) — the call fails by name, the service still serves.

Refusing to boot is right only when the process has no useful behaviour without
the value — rag-core's sheet-sync *worker* keeps its hard gate for exactly that
reason.

Related: [[reference_ci_history_and_dns_are_not_deploy_status]] ·
[[reference_kb_entries_three_blockers]] · [[reference_envdefault_localhost_masks_missing_config]]
