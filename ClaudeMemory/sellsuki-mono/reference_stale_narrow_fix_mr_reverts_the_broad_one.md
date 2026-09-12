---
name: reference_stale_narrow_fix_mr_reverts_the_broad_one
description: "When a broad MR ships the same fix as a still-open narrow one, merging the narrow MR afterwards REVERTS the broad fix — its branch carries the older, smaller version of the additive set/registry; 'redundant' and 'reverting' look identical in the MR list"
metadata:
  node_type: memory
  type: reference
---

Hit 2026-09-11 in `frontend/oc2plus-linecrm-frontend-member`.

Two open MRs fixed the same AuthGuard bug in `src/react/shell/publicRoutes.ts`:

- **!68** (`fix/oc-4501-login-blank-for-anonymous`) — the narrow, purpose-built fix:
  `new Set(['react-probe', 'login'])`
- **!65** (OC-4502 register port) — shipped the same one-line fix *plus* its own
  entry, because the register page hit the identical bug:
  `new Set(['react-probe', 'login', 'register-by-slug'])`

!65 merged first. That made !68 **not merely redundant — a revert**. Its real
diff against the new `develop` was:

```diff
-  new Set(['react-probe', 'login', 'register-by-slug'])
+  new Set(['react-probe', 'login'])
```

i.e. merging the "fix" would have re-broken `/:slug/register`, the page that had
just shipped, by dropping it back out of the public allow-list.

Its spec was worse: it would have **deleted** the regression test !65 added
(`every react-owned route in ROUTE_TABLE has a deliberate public/protected
decision`) — the exact guard written to stop the bug recurring.

**Why this is easy to miss:** in the MR list !68 still reads as "fix for a real
bug, CI green, no conflicts". Nothing in the title, description, or pipeline
says its content is now older than the target. The branch was cut before the
other fix existed, so it faithfully carries the pre-fix world.

**How to apply — before merging any still-open MR that touches a file a
just-merged MR also touched:**

```bash
git fetch origin
git diff origin/<target> origin/<mr-branch> -- <overlapping path>
```

Read that diff **in the direction target → branch**: it is what merging will
actually do. Deletions of things the target gained are the signal. A GitLab
conflict marker is the lucky case; a clean "redundant-looking" diff is the
dangerous one.

Applies to any additive structure several branches append to — allow-lists,
route tables, migration registries, feature-flag sets, DI wiring. Related:
[[reference_fast_forward_merge_silently_reverts]] (same class, different
mechanism — FF drops commits instead of set members),
[[reference_silent_semantic_merge_break]],
[[reference_react_route_owner_flip_needs_public_allowlist]].
