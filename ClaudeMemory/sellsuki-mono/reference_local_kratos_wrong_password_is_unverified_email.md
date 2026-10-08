---
name: reference_local_kratos_wrong_password_is_unverified_email
description: "Local *.sellsuki.local login says \"wrong password\" when the password is RIGHT but the email is unverified (require_verified_address hook) — or when the login flow expired; check Kratos logs before resetting anything"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7fb71c5f-6e71-4687-b858-9caa035598f1
  modified: 2026-10-08T08:44:22.039Z
---

Local Kratos (`backend/kratos-ui-go/kratos_schema_client/kratos.yml`) has `selfservice.flows.login.after.password.hooks: require_verified_address`.
A correct password for an identity whose email is unverified passes the credential check, then the post-login hook refuses it
(400) — and the kratos-ui login page shows it as a wrong password. Seen 2026-10-08 with `nuntawit@sellsuki.com` (identity
`317c3e2a…`, `verified:false`); `nuntnet@gmail.com` (`28c87280…`) is verified.

A login page left open across a Caddy outage/recreate also fails: POST returns 410 "login flow has expired" — reload the page.

**How to check (read-only):** `docker logs --since 30m sellsuki_mono-kratos-1 | grep "08:40:3"`-style around the attempt —
`Running ExecuteLoginPostHook ... identity_id=<id>` followed by a 400 means the password was accepted and a hook refused it.
Look up the identity: `rtk proxy curl -s "http://localhost:4434/admin/identities?credentials_identifier=<email>"` (json.loads strict=False).
**Fix:** verify the address (verification flow; the code lands in mailslurper http://localhost:4436 — [[reference_local_kratos_mail_mailslurper]]),
or log in with a verified identity. Related: [[reference_caddy_host_networking_gotcha]], [[reference_kratos_return_to_allowlist_location]].
