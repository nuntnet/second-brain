---
name: reference_bola_follower_metadata_two_stores
description: BOLA follower metadata lives in contact_profiles (authoritative) + mirror columns on the followers table; ResolveFollowerProfiles is the only place that decides which wins
metadata:
  type: reference
---

**Table naming trap first:** there is no `customers` table. The Go package
`customer`, struct `customerModel`, and folder `customer_repository` all map to
the table **`followers`** (`TableName()` at
`src/repository/customer_repository/postgres_gorm_model.go:90`). `custom_fields`
exists on exactly two tables — `followers` and `contact_profiles` — both `text`
holding JSON. `phone_contacts` has **no** `custom_fields`.

`contact_profiles` is workspace-scoped with **no `line_oa_id`**, so a person's
metadata is deliberately shared across every OA in the workspace. `followers` is
per-OA (`line_oa_id`), so one human following two OAs has two follower rows. The
junction `phone_contact_followers` carries `line_oa_id` + `follower_id`.

**Which store wins:** `contact_profiles` since migration 0072. The follower
columns (`email`, `note`, `tags`, `custom_fields`) are a **mirror**.

**The single decision point is `UseCase.ResolveFollowerProfiles`**
(`src/use_case/follower_profile_resolution.go`, added by MR !177 on 2026-09-10).
Order: follower's own profile → the linked phone_contact's profile → the mirror.
Its SQL twin is `src/repository/customer_repository/effective_metadata_sql.go`
(`effectiveCustomFieldsExpr`, `effectiveEmailExpr`) so search resolves like
display. **Any new follower read that exposes metadata must go through one of
those two** — before !177 the overlay was a per-caller decoration and six readers
silently used the mirror (rich-menu rules, follower CSV export, outbound
webhook, both single-follower enriched routes, the attribute lookup, and the
registration-form write).

**Traps**
- Never mutate the map returned by `GetContactProfilesByFollowerIDs` — an existing
  test caught this: it aliased the mock's return value and made the enrichment
  running afterwards report a phone-contact-owned profile as the follower's own.
- The SQLite unit-test dialect has no `contact_profiles`, so every SQL predicate
  needs an `isSQLite` branch (46 such branches already exist in
  `customer_repository` + `segment_repository`).
- The suite cannot detect a reader using the wrong store: mocks return whatever
  the test primed. Prove such a fix by deleting it and watching the test fail.

Still open: the mirror columns are written, not dropped. The drop (BOLA-272 R3) is
mechanical after !177 but gated on a production-verified backfill +
`bola_contact_dualwrite_divergence_total = 0` (BOLA-284). Separately, a
registration-form field of type `phone` still does not upsert/link a
`phone_contact` — `UpdateFollower` does it correctly and is the model to copy.

Related: [[project_bola_contact_profile_model]] · [[project_bola_is_enabled_int_bool_mismatch]]
