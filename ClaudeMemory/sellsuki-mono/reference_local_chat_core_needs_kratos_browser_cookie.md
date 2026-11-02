---
name: reference_local_chat_core_needs_kratos_browser_cookie
description: Local chat-core (:8099) authenticates by forwarding the Cookie header to Kratos whoami — X-User-Id headers and API-flow session tokens get 401 invalid_session; log the seeded verify identity in through the Kratos BROWSER flow on :4433 and send its sellsuki_session cookie
metadata:
  node_type: memory
  type: reference
  originSessionId: 4a70053d-20ab-45c5-8ae9-5097398e8ddc
  modified: 2026-11-02T20:32:01.558Z
---

Seen 2026-11-02 (AI-374), capturing real samples for fixtures. chat-core's `middleware/sessionauth` strips client
X-User-Id/X-Company-Id and verifies `c.Get("Cookie")` with Kratos, so the sessions.md "Go APIs: curl with X-User-Id"
route does not work for chat-core, and a token from `/self-service/login/api` is rejected too.

**How to apply:** GET `http://localhost:4433/self-service/login/browser` (Accept: json) with a cookie jar → take
`csrf_token` from `ui.nodes` → POST the flow action (rewrite its host to `localhost:4433`) with
`{method:"password", identifier, password, csrf_token}` + the jar cookies → read `Set-Cookie: sellsuki_session=…` →
curl chat-core with `-H "Cookie: sellsuki_session=…" -H "X-Company-Id: <company>"`. Credentials come from the gitignored
`.claude/verify/.local/credentials.json` (local test values; never print them). Needs unsandboxed Bash. Pipe curl to a
file (`-o`), not through rtk, when the body must be parsed. Related: [[reference_rag_staging_probes_user_mints_bff_token_port_forward_ok]].
