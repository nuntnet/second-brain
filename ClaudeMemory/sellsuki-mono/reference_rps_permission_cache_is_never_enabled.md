---
name: reference_rps_permission_cache_is_never_enabled
description: "rps has a Redis permission cache in code but the server builds it with a nil client — nothing is ever cached, so grants and revokes take effect immediately; verified end to end 2026-09-25"
metadata:
  type: reference
---

`ketoPermissionRepository.CheckPermission` (rps `src/repository/permission_repository/keto_permission.go`)
has a full Redis cache with a 5-minute TTL for both allowed and denied answers.
**It is never turned on.** `cmd/generics_server/helper.go` builds the repository
with `NewKetoPermissionRepository(ketoReadUrl)`, which passes `redisClient = nil`,
and every cache branch is guarded by `if r.redisClient != nil`.
`NewKetoPermissionRepositoryWithCache` has zero callers under `cmd/` on both
`origin/develop` and `origin/main`.

**Verified end to end, not just read (2026-09-25):** against the running local rps
(gRPC :9998, real Keto, real Redis) — create role → assign → check = allowed →
unassign → check **immediately = denied**; and the reverse, denied → assign →
immediately allowed. `redis-cli --scan --pattern '*permission_cache*'` = 0 keys
before, during and after.

**Why this matters:** I opened AI-249 as a security bug ("a revoked user keeps access
for 5 minutes") from reading the cache code alone, and it went into the card, MR !139,
epic AI-248 and the plan artifact before an end-to-end run disproved it. A cache (or
any feature) that exists in code is not a cache that runs. Before claiming behaviour
from a component, find where it is constructed and check what it is given.

If anyone ever wires the cache on, the invalidation in rps !139 is what keeps revokes
immediate — so that MR is latent insurance, not a fix.

Related: [[reference_preset_copy_vs_shared_role]] · [[feedback_verify_absence_claims]]
