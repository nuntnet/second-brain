---
name: reference_ai_chat_local_env_keys_missing
description: Four env keys the AI-chat stack needs on local that exist in code but in no .env or .env.example — each absence is a silent 404/503 that makes the admin console look broken rather than misconfigured
metadata:
  type: reference
---

The AI-chat stack's most common local failure is **a config key that exists in
code but in no `.env` and no `.env.example`**. Four found in one evening
(2026-09-17), each producing a status code that points at the wrong layer.

| service | key | local value | what its absence does |
|---|---|---|---|
| chat-core | `ORY_KRATOS_PUBLIC_URL` | `http://localhost:4433` | every authenticated `/v1` → 503 `session_verifier_unavailable` |
| chat-core | `ROLE_PERMISSION_SERVICE_GRPC_URL` | `localhost:9998` | `/v1/me/workspaces` + `/v1/me/companies` → 503; console lists nothing |
| messaging | `CHAT_FB_SHARED_APP_ID` | `235756003289739` | `/v1/workspaces/{id}/facebook-connection` → **404** |
| messaging | `CHAT_FB_SHARED_APP_OWNER_COMPANY_ID` | `company-fb-pilot` | same — both are needed together |

Ports that are easy to get wrong: **4433** is Kratos' own public API (4455 is
kratos-ui-go, the login UI, which 404s `/sessions/whoami`); **9998** is rps'
gRPC port (9999 is its HTTP port).

## Why the shared-app pair matters more than it looks

`chat.fb_app` registers the pilot Facebook app under company
**`company-fb-pilot`** (a placeholder string), but a page bound through the
console writes the binding with the **real** company UUID. `connectionApp`
only substitutes the platform owner when `appID == sharedFacebookAppID` — so
with the pair unset, the lookup is `(real company, app)`, finds nothing, and
the console's Facebook page answers 404 for every real workspace. The user
cannot reconnect a page through the UI at all.

## The shape to recognise

Two of the four are **present but blank** rather than missing, and the
monorepo's `ensure_env_key` backfill skips a key that is merely present — so a
blank line stays blank forever. `grep -c '^KEY='` on the `.env` answers
"is it there" without exposing the value (reading `.env` is hook-blocked).

Related, same family: [[reference_chatcore_missing_kratos_url_503]],
[[reference_procfile_messaging_disables_chat_module]].
