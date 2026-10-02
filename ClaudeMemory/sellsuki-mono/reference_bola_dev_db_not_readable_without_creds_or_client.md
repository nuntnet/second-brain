---
name: reference_bola_dev_db_not_readable_without_creds_or_client
description: Reading BOLA dev data (follower / contact_profile) from a session — kubectl works but there is no SQL client and no read endpoint on the system token; what each route would cost
metadata:
  type: reference
---

Checked 2026-10-02 09:58 (after the 09:00 wake-up, `kubectl` on the staging-th context worked): ns `bola-dev`, pod `back-office-of-line-api-backend-*` 1/1.

- The pod image is busybox-only: `wget` yes, **no psql, no curl**. No psql on the laptop either.
- `DATABASE_*` and `SYSTEM_ADMIN_TOKEN` are `secretKeyRef` (`bola-backoffice-secret`) — reading them out to a shell is credential use that needs the user's say-so.
- The system token (`SystemAdminGuard` / `MachineOrAdminGuard`) reaches only `POST /v1/contacts/link` (writes) and `/v1/system/*` (workspace admin). **No read of follower or contact_profile.** Everything else is `FlatAdminGuard` (admin cookie).
- The pod was ~44 min old, so its logs cannot show last night's registration/link calls.

**Cheapest honest check without credentials:** the user opens the contact in the BOLA console. The detail page goes through `ResolveFollowerProfile` — the same resolver the rich-menu rule uses — so "oc2plus_id visible there" means the rule sees it.

Other routes (both need explicit approval): a temporary read-only psql pod fed by `secretKeyRef` (creates a resource), or pulling the secret locally.

**Context drift (2026-10-02 13:53):** `kubectl config current-context` had flipped to the **production** cluster between two of my calls (another session or the user). Always pass `--context teleport.internal.staging-th.sellsuki.com-teleport.internal.staging-th.sellsuki.com` explicitly for dev-th work; never rely on the current context.

**Secret-equality check without printing:** `kubectl exec ... sh -c 'printf %s "$VAR" | sha256sum'` in both pods and compare the first 10 hex chars plus `wc -c`. Verified this way that member-api `BOLA_API_KEY` == BOLA `SYSTEM_ADMIN_TOKEN` on dev (same hash, 30 chars).
