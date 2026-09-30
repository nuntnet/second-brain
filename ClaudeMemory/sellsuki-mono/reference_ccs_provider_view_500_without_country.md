---
name: reference-ccs-provider-view-500-without-country
description: CCS GET /v1/provider/{code} 500'd when the provider row had no country — CCS2 then cannot open at all; local seed-dev created exactly that (fixed 2026-09-30, CCS !382 develop / !383 main); dev/staging providers unchecked
metadata:
  type: reference
---

**Symptom:** `provider.sellsuki.local` (CCS2) stuck on "Failed to load provider", login irrelevant.
`GET /v1/provider/sellsuki` → 500 `rpc error: code = Unknown desc = record not found`.

**Cause:** `enrichProviderData` (src/use_case/generic_logic.go) asked address-backend for the
provider's country and returned the error. `providers.address_country_code` was empty because
`scripts/seed-dev.sh` inserted `(company_code, name)` only. address-backend's country repo
returns a raw gorm `ErrRecordNotFound` → gRPC `Unknown`, so **not-found and an outage cannot be
told apart by code** — do not string-match the message. The two list callers also filter on
`err == nil`, so such a provider was silently missing from provider lists.

**Fix:** skip the lookup when the code is empty (nothing to ask); a set-but-failing code still
errors. Local: seed-dev now seeds THA and backfills (`4abe44e2`). CCS MRs: !382 (develop),
!383 (main) — `enrichProviderData` was byte-identical on both lines, cherry-pick applied clean.

**Unverified:** whether any provider on dev/staging has an empty `address_country_code`
(both are off 21:00–09:00; this was worked at 07:40). If one does, CCS2 is broken there for
this reason. Check: `SELECT company_code FROM providers WHERE coalesce(address_country_code,'')='';`

**Same shape, not fixed:** `enrichCompanyData` also returns the address-country error (company
creation validates a country, so a row without one is unlikely — but it is the sibling).

See [[reference-central-config-develop-lacks-main-seeds]] for the redact-hook false positive that
blocks a NEW CCS branch (kimzey's fixture on develop) — verify your diff is clean, then
`GSTACK_REDACT_PREPUSH=skip`.
