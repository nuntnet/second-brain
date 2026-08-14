---
name: project-oc2plus-customer-app-auth-plan
description: "OC2Plus Customer App epic (OC-4344) auth architecture decision: self-build device-trust/password/OIDC on member-api instead of Better Auth or a second OIDC Provider; 3 new cards OC-4445/4446/4447 added, blocked by spike OC-4345"
metadata:
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-14T03:07:37.498Z
---

Epic OC-4344 "Customer App" replaces posh-medica (a 3rd-party team's frontend that calls `3rdparty-api` directly from the browser) with a first-party multi-surface app (LIFF / mobile-web / desktop) built on the existing `oc2plus-linecrm-frontend-member` + `member-api` (Go/Fiber, the intended sole BFF — browser never calls `3rdparty-api` directly).

**Auth decision reached in this session** (grounded in actual code, not just the Jira card text):
- `member-api` already issues session cookie `oc2plus_crm_session` (opaque token, Postgres-backed `session` table, fixed 24h TTL, no sliding renewal) for LIFF login, and has working OTP infra (Thaibulk, `otp_sessions` table, lockout via `FailCount`/`LockedUntil`) from OC-4245.
- User asked to add: remember-device (reduce repeat OTP), username/password, and OIDC login (Google/Apple). All three approved and turned into new cards **OC-4445 / OC-4446 / OC-4447** under epic OC-4344, each `blocked by OC-4345` (the auth/BFF spike, still To Do as of 2026-08-14).
- **Rejected Better Auth** — it's a Node/TS-only framework, `member-api` is Go; no viable integration path without either rewriting member-api or standing up a second Node BFF (exactly the anti-pattern OC-4345 is trying to avoid).
- **Rejected building member-api into an OIDC Provider** — Ory Hydra already fills that role in the org, live and wired via `backend/kratos-ui-go` (`share-dev`/`share` namespaces). What OC-4447 needs is a Relying-Party (redirect+callback to Google/Apple) flow instead — genuinely new to the org (no RP flow exists anywhere in the monorepo yet), but additive inside `member-api`, no new service.
- Plan: reuse existing primitives — `GenerateOpaqueToken` (`unique_repository/local.go:20`) for the new device-trust token table, the existing OTP lockout pattern for password lockout, and promote `golang.org/x/crypto` (bcrypt) from indirect to direct dependency. `coreos/go-oidc` has precedent in `backend/bola-backend` (as a bearer-token verifier, different role) so it's not an unfamiliar library for the org.

**Why this matters:** if a future session is asked "should we use $AUTH_LIBRARY for OC2Plus customer login", the answer should reference this decision (self-build, Go-only, no external framework) rather than re-litigating Better Auth from scratch — same class of risk the repo's own `design-doc-authority` rule warns about (contradicting decisions both getting implemented).

Full writeup published as an Artifact (URL not durable across sessions — regenerate via Jira OC-4344/4345/4445/4446/4447 if needed).
