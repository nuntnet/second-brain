---
name: project-bola-ops-visibility-ccs1
description: "Cross-workspace BOLA ops visibility design (BOLA-288/289/290) — home is CCS1 (sellsuki-system-management-frontend) not CCS3, because self-service BOLA workspaces have no company"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-07-21T12:34:29.540Z
---

Sellsuki ops/support needed a system-wide view of BOLA workspaces + admins, across companies AND self-service/standalone signups. First design (CCS3 `bola_workspace_admins` ledger, from [[project_oc_bola_domain_boundary]]/OC-4384-86 work) was **wrong** — that ledger only ever contains company-bound workspaces, so self-service workspaces (`CompanyID == ""`) would be silently invisible.

**Correction (user-driven):** the right home is **CCS1** — `frontend/sellsuki-system-management-frontend`, the existing internal "system admin" panel (AMS-cookie auth, i18n/schema/shipmunk config, zero notion of "company") — NOT CCS3 (`sellsuki-central-control-backend`, company management). "CCS1" is the team's informal name for this system-wide, non-tenant-scoped admin tool (see Jira tag `[CCS1]` on old PAT-682).

**Final design (BOLA-288 epic, BOLA-289/290 stories):**
- Reuse bola-backend's EXISTING `X-System-Token`-gated `/v1/system/workspaces` + `/v1/system/workspaces/:id/admins` — queries BOLA's own DB directly, inherently covers both company-bound and self-service. No new BOLA endpoint needed.
- New thin proxy in `backend/central-configuration-system` (already serves this frontend, already trusts `X-User-Id`/`X-User-Kind` from the AMS gateway) holds `BOLA_SYSTEM_TOKEN` server-side, never sent to browser.
- **bola-backend is deliberately NOT wired behind the AMS/X-User-Id gateway** — Caddyfile confirms `bola.sellsuki.local` doesn't inject `X-User-Id` unlike `pis-app`/`pis.sellsuki.local`. This is intentional self-host isolation (BOLA must have zero runtime dependency on platform identity/Keto/CCS infra) — do not "fix" this to make an ops tool easier.
- Permission gate v1 = static email allowlist config on the proxy (`OPS_ALLOWED_EMAILS`) — the `identity.Identity{Id, Kind}` header contract (shared across central-configuration-system/shipmunk-go/i18n-management-backend) has NO role/permission field, only broad actor `Kind` (e.g. `entity.SellsukiUser`). Real Keto namespace (`sellsuki.internal:ops_support`) explicitly deferred until a second consumer justifies it.

Full design doc: `docs/bola-cross-workspace-ops-visibility.md`.

**Why:** avoids rebuilding the CCS3 ledger's blind spot in a second tool, avoids compromising BOLA's self-host isolation guarantee, avoids over-building Keto RBAC for a single-page internal tool with no second consumer yet.

**How to apply:** any future "ops needs to see across all BOLA workspaces" request should extend BOLA-289/290, not the CCS3 ledger. Any request to add BOLA behind the AMS gateway should be checked against the self-host isolation rule first.
