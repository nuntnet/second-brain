---
name: reference_local_kratos_identity_debugging
description: "how to find WHICH local Kratos identity a real browser session actually resolves to — admin API guessing is unreliable, a backend service's own request logs are ground truth"
metadata:
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-26T04:23:13.373Z
---

When a local `*.sellsuki.local` browser session needs a specific identity (e.g. to grant it an RPS permission), do **not** try to guess it from Kratos's admin API:

- `GET /admin/identities` (port 4434) lists every identity ever created locally (often 10+ stale test/throwaway accounts) — picking "the one with a real-looking email" or "the most recently created" is a coin flip. Confirmed wrong twice in one debugging session this way.
- `GET /admin/identities/{id}/sessions?active=true` is *not* reliable either — it can show a session for the wrong identity (e.g. one created by an API-testing tool, `user_agent: "OpenAPI-Generator/1.0.0/go"`, unrelated to the real browser) while the actual logged-in browser identity shows zero sessions via this same query.
- Querying `POST /sessions/whoami` (port 4433) directly with a session-cookie value the user pastes from DevTools is fragile too — copy/paste through chat can silently corrupt a long token, producing a spurious 401.

**What actually works:** check the request logs of *any* backend service the browser is already successfully authenticating against through Caddy's `forward_auth` (e.g. `oc2plus-line-crm-service-backoffice-api`'s structured JSON logs record the resolved `X-User-Id` per request). Capture via `tmux -L <overmind-socket> capture-pane -t <service> -p -S -N | grep -i "X-User-Id"` and use that exact identity — it's the one Caddy is actually forwarding for the live session, not a guess.

**How to apply:** any time you need "the identity behind this local browser session" (to grant a role, check ownership, etc.), go straight to a backend's own access logs first — treat Kratos admin lookups as a last resort, not a starting point.
