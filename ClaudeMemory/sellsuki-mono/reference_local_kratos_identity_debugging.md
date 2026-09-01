---
name: reference_local_kratos_identity_debugging
description: "how to find WHICH local Kratos identity a real browser session actually resolves to — admin API guessing is unreliable, a backend service's own request logs are ground truth"
metadata:
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-09-01T04:57:25.875Z
---

When a local `*.sellsuki.local` browser session needs a specific identity (e.g. to grant it an RPS permission), do **not** try to guess it from Kratos's admin API:

- `GET /admin/identities` (port 4434) lists every identity ever created locally (often 10+ stale test/throwaway accounts) — picking "the one with a real-looking email" or "the most recently created" is a coin flip. Confirmed wrong twice in one debugging session this way.
- `GET /admin/identities/{id}/sessions?active=true` is *not* reliable either — it can show a session for the wrong identity (e.g. one created by an API-testing tool, `user_agent: "OpenAPI-Generator/1.0.0/go"`, unrelated to the real browser) while the actual logged-in browser identity shows zero sessions via this same query.
- Querying `POST /sessions/whoami` (port 4433) directly with a session-cookie value the user pastes from DevTools is fragile too — copy/paste through chat can silently corrupt a long token, producing a spurious 401.

**What actually works:** check the request logs of *any* backend service the browser is already successfully authenticating against through Caddy's `forward_auth` (e.g. `oc2plus-line-crm-service-backoffice-api`'s structured JSON logs record the resolved `X-User-Id` per request). Capture via `tmux -L <overmind-socket> capture-pane -t <service> -p -S -N | grep -i "X-User-Id"` and use that exact identity — it's the one Caddy is actually forwarding for the live session, not a guess.

**How to apply:** any time you need "the identity behind this local browser session" (to grant a role, check ownership, etc.), go straight to a backend's own access logs first — treat Kratos admin lookups as a last resort, not a starting point.

**Generalizes to staging/dev-th too, not just local.** Same principle, different capture command: `kubectl logs -n <namespace> <backend-pod> --since=15m | grep -oE '"X-User-Id":"[0-9a-f-]{36}"'` against the real backend pod in the target cluster namespace (e.g. `octoplus` = staging-th's OC2Plus namespace on the staging-th Teleport cluster, `octoplus-dev` = dev-th's). Confirmed identity from staging backoffice-api's structured JSON logs this way (`X-User-Id` field, same log shape as local).

**Also learned debugging staging directly:** a real deployed gateway (unlike local Caddy, which has a deliberate curl-testing bypass for an explicit `X-User-Id` header) will **not** honor a client-supplied `X-User-Id`/`X-User-Kind` from outside — it only trusts what it derives itself from a real session/JWT. Hitting the public domain (e.g. `api.staging-th.sellsuki.com`) with a manually-set identity header gets ignored, producing a misleading "unauthenticated" error. To test/debug a staging service *as a specific identity* the way you can locally, `kubectl port-forward` straight to the service's ClusterIP/pod (bypassing the public gateway entirely) and set the header on that direct connection instead.
