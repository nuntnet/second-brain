---
name: project_workspace_is_multi_product_ccs2_support
description: "Workspace" means store + BOLA + chat (not BOLA only); CCS2 needs the CCS3 unified workspace view so provider admins can support companies — OC-4263 (BOLA-only) is outdated
metadata:
  type: project
---

nunt 2026-11-22: a company's workspaces are several products (store, BOLA, chat — more may come). CCS3 already manages them
together (AI-284/285/286); CCS2 must offer the same per company so a **provider admin can support companies**.
OC-4263 / OC-4291 were written 2026-06 as BOLA-only and are wrong on that point.

State 2026-11-22: CCS unified read model + member routes (`GET /companies/{id}/workspaces`, `.../{type}/{id}/members|roles|invitations`)
are on CCS main AND develop. CCS3 unified page (AI-286) is on CCS3 **develop only**. CCS2 still has separate `stores` and
`bola-workspaces` tabs on the old endpoints. Draft of the corrected card: `docs/cards/drafts/oc4263-workspaces-2026-11-22/CARD.md`.

**Why:** a card that says "BOLA workspace" gets built BOLA-only and chat support is missing for provider admins.
**How to apply:** any CCS2/CCS3 workspace card covers every `ManagedWorkspaceType`, reads the unified API, and drives actions
from server `capabilities`. Related: [[project_rbac_preset_backfill_cards_pat2872_2873]].
