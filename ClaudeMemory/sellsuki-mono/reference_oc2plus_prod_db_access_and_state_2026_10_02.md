---
name: reference_oc2plus_prod_db_access_and_state_2026_10_02
description: "OC2Plus production — how to read the CRM DB (RDS reachable directly, secret key names differ from the env names, psql is keg-only), what prod runs (images from Nov 2025–Feb 2026), and the migration ledger state read on 2026-10-02"
metadata:
  node_type: memory
  type: reference
  originSessionId: 26a7f96a-8890-4fad-a7f0-689c6c35e9d5
  modified: 2026-10-02T09:58:43.746Z
---

**Access (read-only, user runs it — Claude does not pull prod credentials without explicit yes):**
- Teleport `production-cluster`, namespace `octoplus`. Secret `oc2plus-crm-secret`. The Deployment env names do NOT match the secret keys: `POSTGRES_USER` ← `POSTGRES_USERNAME`, `POSTGRES_PASS` ← `POSTGRES_PASSWORD`, `POSTGRES_CRM_DB_NAME` ← `POSTGRES_CRM_DATABASE`. Reading the wrong key returns an empty string, not an error.
- DB is RDS `oc2plus-production.ck8kw4gbqop4.ap-southeast-1.rds.amazonaws.com`, db `crm_production`, user `crm_service`. Reachable straight from the laptop on 5432 (no port-forward). `tsh db ls` is empty.
- `psql` = `/opt/homebrew/opt/libpq/bin/psql` (libpq is keg-only; each terminal tab needs the PATH export). Use `PGOPTIONS='-c default_transaction_read_only=on'` and `-P pager=off` (the pager truncates output). Unset PG* vars afterwards.
- `member` PK is `member_id` (not `id`); `member_point_activity.member_id`; `member_identity.identity_id`.

**What prod runs (2026-10-02):** member-api v1.4.0 (2025-11-13), 3rdparty-api v1.8.0 (2025-11-24), backoffice-api v1.11.0 (2026-02-11) — `main` is 283/144/446 commits ahead. The first prod release is a ~10-month catch-up, not a sprint.

**Migration ledger (`migrations`, 76 rows):** last applied `20260121040652-create-table-invitation`. All 55 repo-530 migrations from `20260630000004` onward are pending; none of the 29 tables they create exist. 10 ledger names are not in repo 530 (broadcast/richmenu/file-storage/invitation…) — another source writes this ledger. `name` values start with `/`.

**Blocker found:** `20260710035709-add-member-company-phone-unique` fails — 2 duplicated (company_id, phone) groups (companies `5e102841-…` ×3, `c88e62ca-…` ×2, different LINE identities). OC-4469 requires that index. Owner of the data not yet decided.

Written to Jira OC-4536 (comments 45539, 45546) and `docs/release/environments.yaml` (prod). See [[reference_migration_files_are_not_applied_schema]] and [[reference_oc2plus_schema_lives_in_external_repo]].
