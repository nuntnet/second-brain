---
name: reference_chatcore_missing_kratos_url_503
description: chat-core's local .env has no ORY_KRATOS_PUBLIC_URL, so every authenticated /v1 request answers 503 session_verifier_unavailable and the admin console loads nothing — while an unauthenticated curl still returns a healthy-looking 401
metadata:
  type: reference
---

`sessionauth` (chat-core, merged 2026-08-25) verifies every `/v1/*` browser
request against `GET {ORY_KRATOS_PUBLIC_URL}/sessions/whoami`. The key is in
**neither** chat-core's `.env.example` **nor** the monorepo's
`scripts/setup.sh`, so no local stack has ever had it.

Unset is not a degraded mode. The console shows:

```
GET https://chat.sellsuki.local/v1/me/workspaces  503
GET https://chat.sellsuki.local/v1/me/companies   503
{"error_code":"session_verifier_unavailable"}
```

— no workspaces, no companies, nothing.

## Why it eats an afternoon

**The service looks healthy from outside.** A request with no cookie
short-circuits to `401 invalid_session` *before Kratos is ever called*, so
curl, health probes and smoke checks all pass. Only a browser holding a real
session ever reaches the broken call.

The discriminator is a **garbage cookie**:

```bash
curl -s -H 'Cookie: ory_kratos_session=garbage' localhost:8099/v1/me/workspaces
```

`401 invalid_session` = verifier reaches Kratos, fine.
`503 session_verifier_unavailable` = it cannot, and this is your bug.

## Fix

```bash
ORY_KRATOS_PUBLIC_URL=http://localhost:4433   # in backend/sellsuki-chat-core/.env
```

**4433 is Kratos' own public API, NOT 4455** — `kratos-ui-go` is the login UI
and answers 404 for `/sessions/whoami`. That same confusion previously caused
an infinite login-redirect loop in the admin console.

Landed as chat-core MR !86 (`.env.example`) and a monorepo `ensure_env_key`
call in `scripts/setup.sh` that backfills existing stacks.

Related trap, same shape — a lower layer answering an unexpected status while
everything above looks fine: [[reference_procfile_messaging_disables_chat_module]].
