---
name: project-test-users-as-infrastructure
description: "The user's recurring blocker \"the agent has no user to test with / stuck at OTP\" and the 2026-10-11 durable answer — scripts/seed-verify.sh + .claude/verify/.local/credentials.json + sessions.md; the agent never asks a human for a local login; what the seed cannot create"
metadata:
  node_type: memory
  type: project
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-10T18:15:37.144Z
---

2026-10-11 the user named a long-standing problem: when verifying, the agent lacks a
user to test with, or is stuck at an OTP, so pstack-style end-to-end verification
never completes. They asked for a long-term fix, not a workaround.

**The design (built):** test identities and sessions are infrastructure.
- `scripts/seed-verify.sh` (idempotent, localhost only): creates a Kratos identity with
  password **and verified email** via the admin API (kratos-ui-go refuses unverified
  as "wrong password"); `--grant` runs seed-dev with that id for CCS roles/company;
  `--fix-env` writes `VITE_LOCAL_DEV_IDENTITY_ID` (backoffice bypass) and
  `VITE_LOCAL_DEV_TEST_SECRET` (= member-api `TEST_KEY`) into the gitignored
  `.env.development.local` files; `--bola` creates a workspace via the system API.
- Credentials land in `.claude/verify/.local/credentials.json` (gitignored); the
  `verifier` agent and `/verify` read it and never ask a human for a local login.
- `.claude/verify/sessions.md`: per-surface recipe (Kratos password login for
  CCS3/BOLA/backoffice-AMS; Vite bypass for backoffice local API; seed phone + OTP
  `000000` for the member app; `X-User-Id` header for Go APIs; LIFF unreachable).

**Why:** pstack's whole verification story (Control Glass, verifier before merge)
assumes the agent can drive the real app; without a seeded user every live pass ends
in `verified-unreachable`.

**How to apply:** before any live drive on local, run `scripts/seed-verify.sh` and
read its report; treat a refusal as a prerequisite to name, not a question to ask.
Never store or request dev/staging/prod credentials this way. Known gap: Route B
session minting for the member app needs `integration` + `member_identity` rows the
seed does not create yet. Related: [[reference-member-app-local-test-mode-not-active]],
[[project-pstack-as-workflow-benchmark]].
