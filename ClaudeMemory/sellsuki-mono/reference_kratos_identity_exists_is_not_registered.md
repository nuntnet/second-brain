---
name: reference_kratos_identity_exists_is_not_registered
description: In Kratos, an identity existing (or having a password credential entry) does not mean the person can sign in — only config.hashed_password via include_credential proves it
metadata:
  type: reference
---

Deployed Kratos is **v25.4.0** (`docker-compose.yml`). Probed directly against the local admin API on 2026-09-09:

`POST /admin/identities` with traits and no `credentials` creates an identity that **cannot log in**. Every BOLA invite that was never completed leaves one of these behind under that email — which is also why the second create returns 409.

The trap: **a traits-only identity and one created WITH a password are indistinguishable in the default read.** Both report `credentials: {password, webauthn}` — Kratos writes the identifier row for uniqueness whether or not a secret was ever set — and both have `config` redacted.

| read | traits-only | with password |
|---|---|---|
| `GET /admin/identities` (list) | no `credentials` key at all | no `credentials` key at all |
| `GET /admin/identities/{id}` | `{password, webauthn}`, config redacted | `{password, webauthn}`, config redacted |
| `…?include_credential=password` | `config: {}` | `config: {hashed_password: "$argon2id$…"}` |

So the only reliable check is `credentials.password.config.hashed_password` **with** `include_credential` (repeated params work: `?include_credential=password&include_credential=oidc`, 200). Count `oidc` too — an SSO user has no password. Do **not** count `webauthn`: it is a second factor here, not a way in.

Branching on mere existence activates a member who can never sign in, and **no error is raised anywhere** — a silent lockout.

Also: this environment's schema id is `email`, not `default`, and it **requires** `ref_id`, `phone` and `pdpa_reference_id` traits (a create without them 400s). `phone` needs a real 10-digit Thai mobile shape.

Implemented as `kratosAuthProvider.identityCanSignIn` in bola-backend (MR !170) — see [[project_bola309_lane2_add_existing_member]].
