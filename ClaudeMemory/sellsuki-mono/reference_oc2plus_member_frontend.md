---
name: reference_oc2plus_member_frontend
description: "OC2Plus customer member-registration LIFF frontend = existing repo oc2plus-linecrm-frontend-member (GitLab, 2yr); OC-4245 belongs there, not a new repo"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

The customer-facing OC2Plus **member registration LIFF frontend** already exists on GitLab as `git@gitlab.sellsuki.com:sellsuki/oc2plus/line-crm/frontend/oc2plus-linecrm-frontend-member.git` — 2+ years old, active. Branches confirm it IS the LINE member-register app: `feature/oc-2396-member-register-line-account`, `enhance/oc-2782-change-text-message-register-line`, `bug/oc-2429-check-member-activation`, `enhance/posh`, `develop`.

⚠️ It is NOT added as a submodule in this monorepo by default — `frontend/` only had `oc2plus-linecrm-frontend-backoffice` (the Vue3 admin). **Checking local monorepo submodules is NOT enough to conclude "no customer frontend exists" — always `git ls-remote` the GitLab group too.**

**OC-4245 (membership register/enrich page) belongs in this existing `-member` repo** (likely an enhancement of the existing register-line flow, wiring it to the new OC-4289 Membership API + domain boundary), NOT a greenfield app.

**OC2Plus line-crm has TWO separate backends (both on GitLab, neither was a monorepo submodule):**
- `oc2plus-line-crm-service-backoffice-api` — admin/backoffice API (HTTP 8089), the only one in the monorepo. Has its own DB.
- `oc2plus-line-crm-service-member-api` — **customer-facing member API** = `api.member.oc2.plus` (dev `api.member.dev-th.oc2.plus`). This is what the `oc2plus-linecrm-frontend-member` frontend calls (`VITE_SERVICE_MEMBER_BASE_URL`). Branches: develop, feature/oc-2313-login, enhance/oc-3426-change-endpoint-management-backend-to-ccs, etc. Existing member-create flow: frontend sends `line_access_token` → member-api creates member (LINE-coupled).

⚠️ **OC-4289 (customer Membership API POST /members) belongs in `oc2plus-line-crm-service-member-api`, NOT backoffice-api.** I built it in backoffice-api by mistake (same root cause — only checked local submodules, didn't ls-remote the GitLab backend group). Needs re-homing to member-api + reconciling with its existing member-create flow (which is LINE-coupled vs OC-4289's LINE-agnostic design — a real design decision). BOLA-264 (bola-backend) is correctly placed.

**Lesson (recurring 3x this session):** before assigning a card's repo home, `git ls-remote` the GitLab group (`sellsuki/oc2plus/line-crm/{frontend,backend}/...`) — the monorepo only contains a subset of repos as submodules.

Mistake made 2026-06-26: I scaffolded a NEW repo `frontend/oc2plus-linecrm-frontend-liff` (React18/Vite/@line/liff, port 5183) + wired the monorepo (commit `bfb944d` on `feat/oc-4200-member-follower`) believing no customer frontend existed — because I only checked local submodules. The `-liff` GitLab project never existed (404). Corrected by discarding the scaffold + adding the real `-member` repo as submodule. See [[project_oc_bola_domain_boundary]].
