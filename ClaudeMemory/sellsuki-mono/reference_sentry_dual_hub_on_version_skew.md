---
name: reference_sentry_dual_hub_on_version_skew
description: "Adding @sentry/react next to an exact-pinned @sentry/vue silently creates TWO Sentry hubs — errors are captured into a client that was never init'd and vanish with no warning; pin both SDKs to the same version and verify one @sentry/core in the lockfile"
metadata:
  node_type: memory
  type: reference
---

Hit 2026-09-11 in `frontend/oc2plus-linecrm-frontend-member` (OC-4499, React island
next to the existing Vue app).

`@sentry/vue` was resolved at an **exact** `8.41.0`. Installing `@sentry/react`
without pinning took `8.55.2`, whose `@sentry/core` npm cannot dedupe against
`8.41.0`. Two copies of `@sentry/core` means **two hubs**: the React
ErrorBoundary calls `captureException` on a client that `main.ts` never
initialized, so the event goes nowhere.

**Nothing warns you.** No console error, no build failure, no test failure. The
only symptom is events quietly missing from the dashboard — and you don't notice
absence.

**Fix:** pin the second SDK to the *same exact version* as the first
(`"@sentry/react": "8.41.0"`).

**Verify from the lockfile, not from the install log** — one entry only:

```bash
python3 -c "
import json;d=json.load(open('package-lock.json'))
print([ (k,v['version']) for k,v in d['packages'].items() if k.endswith('@sentry/core') ])"
```

More than one `@sentry/core` row = two hubs, regardless of what the top-level
versions say.

Generalises to any SDK family that keeps shared mutable state in a `core`
package (Sentry, OpenTelemetry): mixing minor versions across sibling packages
splits the singleton instead of erroring. Related:
[[reference_flag_without_enforcement]], [[reference_turbo_cache_crosssession_false_green]].
