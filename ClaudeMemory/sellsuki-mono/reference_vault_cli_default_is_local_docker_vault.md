---
name: reference-vault-cli-default-is-local-docker-vault
description: "nunt's shell has no VAULT_ADDR → vault CLI hits 127.0.0.1:8200 = the monorepo's LOCAL docker Vault; a staging write \"succeeds\" there and staging never sees it. Use staging URL (VPN) or Teleport port-forward + OIDC login"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-16T04:02:38.046Z
---

2026-11-16 (AI-56 `BOLA_SERVICE_TOKEN` for chat-core staging): the first `vault kv patch` printed "version 4"
and staging still had 9 keys — `VAULT_ADDR` was unset, so the CLI used `127.0.0.1:8200`, which is Docker's
local Vault (`lsof -iTCP:8200` → com.docker). Both the read (BOLA token) and the write were local.

Staging Vault:
- `https://vault.internal.staging-th.sellsuki.com` resolves to 10.21.x — needs the company VPN (i/o timeout without it).
- Without VPN: `kubectl --context teleport.internal.staging-th... -n system-security port-forward svc/vault-server-active 18200:8200`
  (run as `exec kubectl …` with run_in_background so it survives), then `export VAULT_ADDR=http://127.0.0.1:18200`,
  `vault login -method=oidc` (works through the port-forward; callback is the CLI's localhost:8250).
- `~/.vault-token` may hold the LOCAL token after a local login → 403 on staging; re-login with OIDC.
- KV v2; `patch` falls back to read-modify-write (warns about the patch capability) and keeps other keys.
- Secrets in ns sellsuki are ExternalSecrets (ClusterSecretStore `vault-backend`, refresh 300s); a sync 2 s after a
  write can miss it → `kubectl annotate externalsecret <name> force-sync=$(date +%s) --overwrite`, then count keys.

**How to apply:** before giving a Vault command, write `export VAULT_ADDR=…` into it explicitly. Never let the
user paste `vault token lookup` output into chat — the `id` line is the token. Related: [[reference-outline-mcp-vpn-blocker]].
