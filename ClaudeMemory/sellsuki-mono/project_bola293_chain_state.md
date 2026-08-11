---
name: project-bola293-chain-state
description: "BOLA-118/293 invite chain: rps + CCS merged & deployed to dev (2026-08-11); left = SPA !15 press, bola tag, keto backfill-then-flip, CCS2 role-code mismatch, kratos-mode password gaps"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T08:30:35.573Z
---

State of the BOLA-118/293 platform-invite chain as of 2026-08-11:

**Landed:** rps !86/!89/!90/!93 (internal invite API + BOLA role presets, migration renumbered 0013→0015 after OC-4246 took 0013 on both mainlines); CCS !264 (role-bearing invitations) + !253 (feature/provider→develop mega-merge) merged and **deployed healthy to dev** (ns sellsuki-dev, image aa296868). bola-backend BOLA-293 code is on main but **NOT in any tag** (latest v1.0.29) → SaaS prod still uses recovery-link invites until a new tag is cut.

**Remaining:** press SPA [!15](https://gitlab.sellsuki.com/sellsuki/sellsuki/frontend/sellsuki-invitation/-/merge_requests/15) (mergeable; also carries the dead-runner CI fix) → tag bola-backend (picks up BOLA-293 + APM logs fix !153) → e2e invite test on dev → **backfill role assignments BEFORE flipping `PERMISSION_PROVIDER=keto`** (never before backfill).

**Gaps decided but not yet carded (from the kratos-consolidation analysis):**
1. CCS !253 changed company role taxonomy to name-resolved presets (Company Owner/Manager/…) — **CCS2 provider-management-frontend still sends legacy "admin"/"member" role codes that no longer exist**; member add/change-role UI needs updating.
2. bola-backend kratos-mode password gaps: `SetAdminPassword`/`ResetAdminPassword`/`CreateAdminWithTempPassword` are NOT gated by TokenIssuer (write useless local hashes in SaaS mode); FE `ForgotPasswordPage`/temp-password UI doesn't branch on `VITE_AUTH_MODE`.
3. **`RemoveAdmin` in bola-backend deletes only the local row — no Keto tuple unassign** → removed members keep permissions once keto flips. Must close before the flip.
4. Self-host safety rule: kratos-mode fixes must branch on auth mode, never delete local_jwt paths (rgb72 uses them); migrations boot-run on both modes.

CCS !253 review also removed CCS's unsafe `bind-by-invitation` endpoint (client-supplied role, no code validation) — `AcceptBolaWorkspaceInvitation` is the only accept path now.
