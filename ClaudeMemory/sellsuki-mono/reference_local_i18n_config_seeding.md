---
name: reference-local-i18n-config-seeding
description: "Local i18n labels go stale (snapshot from 2024) — reseed from staging's PUBLIC i18n API via scripts/seed-i18n-from-staging.sh; i18n + central-config admin APIs need a Keto role on tenant sellsuki.system"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T14:07:40.181Z
---

Two local-dev gaps that make company-management (CCS3) look broken, both fixed by seeding (2026-08-11):

**1. i18n labels are a stale snapshot.** Local `i18n` Postgres DB was seeded once in 2024 and drifts: local had **162** `ccs3` keywords while staging had **694**, so most labels rendered as raw keys. Reseed with `make seed-i18n` (`scripts/seed-i18n-from-staging.sh`) — it pulls staging's **public, unauthenticated** endpoints (`GET https://api.staging-th.sellsuki.com/i18n-management/v1/public/languages` and `/public/keywords/{tag}`; reachable without VPN) and UPSERTs straight into Postgres, because the write API is Keto-gated. `tags` is UNIONed on conflict so a keyword shared by several apps keeps its other tags.
  - Tags that exist on staging: `ccs3` (694), `patona` (671), `ccs1` (2), `ccs2` (2). **`invitation` and `sellercenter` return 400** — they don't exist there; the script skips loudly rather than aborting.
  - Local i18n DB = database `i18n`, tables `keywordcollection` (keyword, description, tags[], lang_value jsonb, default_lang), `language` (code, name, image), `tags` (name_tag). The public payload's `DEFAULT` key is a synthetic copy of the default language's value, not a language code.

**2. i18n + central-configuration admin APIs need a Keto role at tenant `{kind: "sellsuki.system", id: ""}`.** Without it: 403 on i18n keyword writes and on `POST /v1/schema/...`. Consequence chain worth remembering — CCS3 reads namespace `sellsuki-global-user`; if that schema was never registered, `GET /v1/configuration/sellsuki-global-user` **404s**, the config store never sets `hasLoadedConfig`, and `matchCompanyId` in `stores/company/current-company.ts` returns `undefined`, so **no company ever resolves in the UI**. `scripts/seed-dev.sh` now creates that role (grpcurl → rps `localhost:9998` CreateRole+AssignRole, same pattern as the provider-owner block) and registers the schema.
  - Permission strings: `sellsuki.i18n.keyword.*`, `sellsuki.i18n.language.*`, `sellsuki.configsystem.schema.*`, `sellsuki.configsystem.config.*`.
  - `POST /v1/schema/{service}` in central-config **panics (500) when `note` or `updateBy` are omitted** — unguarded pointer deref in `ConvertModelRequestToSchema`; should be a 400. Send all fields.

Related: [[reference-ccs3-frontend-facts]] (the dev-bypass identity that must exist in local Kratos), [[reference-ccs-config-namespaces]] (namespace/key inventory).
