---
name: reference-prod-port-forward-denied-kubectl-get-logs-ok
description: "Production cluster: after the user's tsh login, kubectl get/logs work but port-forward (for grpcurl) is denied by the auto-mode classifier as Sensitive Remote Exec — hand the grpcurl command to the user (2026-11-03)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 5b98ed42-0e86-44b4-98df-e4d0502d2574
  modified: 2026-11-03T08:45:02.874Z
---

On 2026-11-03, with the production Teleport session freshly logged in by nunt
(context `teleport.internal.production.sellsuki.com-production-cluster`):
- `kubectl get deploy -A` (images) and `kubectl logs` worked.
- `kubectl port-forward` to rps prod was refused by the auto-mode classifier ("Sensitive Remote Exec").
Staging port-forward + grpcurl stays allowed.

**How to apply:** for a prod read that needs gRPC/DB (rps migration state, QMS plans), read images/logs
yourself, then give nunt the exact port-forward + grpcurl/psql command ([[user-has-vault-and-prod-access-agent-hands-over-commands]]).
Do not retry the port-forward. Note: rps does not migrate at boot, so an image tag on prod says nothing
about which migrations ran.
