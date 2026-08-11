---
name: reference-local-i18n-config-seeding
description: "Local i18n labels go stale (snapshot from 2024) — reseed from staging's PUBLIC i18n API via scripts/seed-i18n-from-staging.sh; i18n + central-config admin APIs need a Keto role on tenant sellsuki.system"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T14:40:23.482Z
---

Two local-dev gaps that make company-management (CCS3) look broken, both fixed by seeding (2026-08-11):

**1. i18n labels are a stale snapshot.** Local `i18n` Postgres DB was seeded once in 2024 and drifts: local had **162** `ccs3` keywords while staging had **694**, so most labels rendered as raw keys. Reseed with `make seed-i18n` (`scripts/seed-i18n-from-staging.sh`) — it pulls staging's **public, unauthenticated** endpoints (`GET https://api.staging-th.sellsuki.com/i18n-management/v1/public/languages` and `/public/keywords/{tag}`; reachable without VPN) and UPSERTs straight into Postgres, because the write API is Keto-gated. `tags` is UNIONed on conflict so a keyword shared by several apps keeps its other tags.
  - Tags that exist on staging: `ccs3` (694), `patona` (671), `ccs1` (2), `ccs2` (2). **`invitation` and `sellercenter` return 400** — they don't exist there; the script skips loudly rather than aborting.
  - Local i18n DB = database `i18n`, tables `keywordcollection` (keyword, description, tags[], lang_value jsonb, default_lang), `language` (code, name, image), `tags` (name_tag). The public payload's `DEFAULT` key is a synthetic copy of the default language's value, not a language code.

**2. i18n + central-configuration admin APIs need a Keto role at tenant `{kind: "sellsuki.system", id: ""}`.** Without it: 403 on i18n keyword writes and on `POST /v1/schema/...`. Consequence chain worth remembering — CCS3 reads namespace `sellsuki-global-user`; if that schema was never registered, `GET /v1/configuration/sellsuki-global-user` **404s**, the config store never sets `hasLoadedConfig`, and `matchCompanyId` in `stores/company/current-company.ts` returns `undefined`, so **no company ever resolves in the UI**. `scripts/seed-dev.sh` now creates that role (grpcurl → rps `localhost:9998` CreateRole+AssignRole, same pattern as the provider-owner block) and registers the schema.
  - Permission strings: `sellsuki.i18n.keyword.*`, `sellsuki.i18n.language.*`, `sellsuki.configsystem.schema.*`, `sellsuki.configsystem.config.*`.
  - `POST /v1/schema/{service}` in central-config **panics (500) when `note` or `updateBy` are omitted** — unguarded pointer deref in `ConvertModelRequestToSchema`; should be a 400. Send all fields.

**3. CCS3 `/users` needs COMPANY-scope permissions — BOLA super_admin is irrelevant.** The page sets `isForbidden = !(canInvite || canView)` from `GET /v1/checkpermission/each?kind=sellsuki.company&id=<company>&permissions=sellsuki.company.invite.create,sellsuki.user.view` (comma-separated — repeating the param 400s). Companies from older local seeds have **no Company Owner preset at all** (the seeded Sukispace company's only role was "OC2Plus API Key Manager"), so the page is permanently forbidden until a company-scope role is granted. `seed-dev.sh` now creates + assigns one per company. BOLA workspace roles and Sellsuki company permissions are **separate Keto namespaces by design** — being super_admin in BOLA grants nothing in CCS3, and no amount of rps syncing changes that.
  - Valid codes to grant (verified present in rps `permission_lists`): `sellsuki.user.view`, `sellsuki.company.invite.create`, `sellsuki.role.view/create/update`, `sellsuki.company.view/update`. `sellsuki.role.list` and `sellsuki.user.list` do **not** exist.
  - Role admin via grpcurl: `ListRoles` takes `{filter_options:{owner_id,owner_kind}, list_options:{limit}}` — the fields are `owner_id`/`owner_kind`, not a nested `owner`.

**4. ⚠ `psql` may not exist on the host at all** (true on this machine, 2026-08-11) — and every legacy `psql … 2>/dev/null` line in `seed-dev.sh` then silently no-ops, leaving a half-seeded local env with **no error message**. A `psql_db` wrapper (host psql → `docker compose exec -T postgres psql` fallback) was added and new steps use it; the older lines are still unguarded. `brew install libpq && brew link --force libpq` fixes them for real.

Related: [[reference-ccs3-frontend-facts]] (the dev-bypass identity that must exist in local Kratos), [[reference-ccs-config-namespaces]] (namespace/key inventory).
