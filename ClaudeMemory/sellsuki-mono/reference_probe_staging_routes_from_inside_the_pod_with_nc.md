---
name: reference_probe_staging_routes_from_inside_the_pod_with_nc
description: "Sandbox cannot bind a kubectl port-forward and cannot curl *.staging-th hosts, so probe a staging service by `kubectl exec <pod> -- sh -c 'printf \"GET /path HTTP/1.0\\r\\nHost: x\\r\\n\\r\\n\" | nc -w 3 127.0.0.1 <containerPort>'` — BusyBox wget hides error bodies (400/401 come back as 'server returned error'), nc shows them. Vault lookup-self body contains the token itself in data.id — grep only expire_time/ttl/policies, never print the body"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4683d2cf-08fe-4531-bc03-8b12af31b5cb
  modified: 2026-10-28T06:24:29.690Z
---

Worked 2026-10-28 on messaging-backend (container port 80; svc is `sellsuki-messaging-backend-svc`, not `sellsuki-messaging-backend`). `kubectl` itself needs `allowed_domains: teleport.internal.staging-th.sellsuki.com:443`. `tsh status` first: the staging-th profile and the production profile expire separately, and an expired production profile is fine (never touch it from a staging task anyway).

For a Vault check from inside the pod: `wget -qO- --server-response --header="X-Vault-Token: $VAULT_TOKEN" "$VAULT_ADDR/v1/auth/token/lookup-self"` then `grep -o '"expire_time":"[^"]*"'` etc. on the saved body, and delete the body — `data.id` IS the token.

**How to apply:** for route probes that must show the response body (which error code, which message), use the nc one-liner inside the pod; for liveness-only checks wget is enough. Related: [[reference_staging_chat_core_replay_recipe_in_pod_wget]], [[reference_sandbox_blocks_go_cli_tls_and_ssh]].
