---
name: reference_verifier_session_exec_and_port_forward_denied_even_with_relayed_approval
description: In a side/verifier session the auto-mode classifier denies kubectl exec AND unsandboxed kubectl port-forward; approval the user gave in the parent session and relayed by message does not lift it — only a prompt approved in that session does. Read-only kubectl get/logs/jsonpath works with allowed_domains teleport.internal.staging-th.sellsuki.com:443
metadata:
  node_type: memory
  type: reference
  originSessionId: f4e6a7ce-8ac3-449f-a071-c829a2a83d39
  modified: 2026-10-29T07:07:14.554Z
---

Seen 2026-10-29 (verify-g3 sweep): `kubectl -n sellsuki get/logs` work sandboxed when `allowed_domains: ["teleport.internal.staging-th.sellsuki.com:443"]` is set. `kubectl exec` and `kubectl port-forward` (dangerouslyDisableSandbox) were both denied twice, including after the parent session relayed the user's "อนุญาต".

**How to apply:** plan a verifier sweep around logs/get/env-name reads plus `glab api`; do not promise live route probes. Tell the user to approve the prompt in the verifier session itself, or hand over the exact command. `kubectl logs` output must be read via `rtk proxy` (rtk rewrites grep output). Related: [[reference_probe_staging_routes_from_inside_the_pod_with_nc]].
