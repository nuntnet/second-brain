---
name: reference_member_api_service_urls_were_never_set
description: "member-api's THIRDPARTY_SERVICE_URL and FILE_SERVICE_URL were set in NO environment — the localhost envDefault made the pod dial itself and the 502 blamed a healthy 3rdparty-api"
metadata:
  type: reference
---

2026-09-22, staging: `GET /v1/me/consent/pdpa/status` returned 502. member-api's
own log said what was wrong, and it read like someone else's outage:

```
thirdparty_repository.GetConsentStatus
Get "http://localhost:8080/consent/pdpa/status": dial tcp [::1]:8080: connection refused
```

`THIRDPARTY_SERVICE_URL` carried `envDefault:"http://localhost:8080"` and was
listed in **none** of `deployment/values-{development,staging,production}.yml`.
`FILE_SERVICE_URL` (point-claim receipt upload, OC-4362) was missing the same
way in all three. `CONSENT_ENDPOINT`, `CCS_SERVICE_BASE_URL` and
`MESSAGING_SERVICE_BASE_URL` were all set — so the file looked complete.

This is the failure mode [[reference_envdefault_localhost_masks_missing_config]]
describes, seen a second time: config parsing succeeds, nothing is nil, no file
is absent, and the pod quietly calls itself. The error surfaces as a *network*
error naming the callee, which sends the reader to a service that is healthy.

**Values that are correct (verified against the staging/dev clusters):**
- 3rdparty-api: `http://oc2plus-line-crm-service-3rdparty-api-svc.<octoplus|octoplus-dev>/v1`
  — 🔴 the `/v1` is load-bearing. Its routes mount at `/v1`
  (`fiber_server.go:79`) and member-api concatenates the path straight onto the
  base (`h.baseURL+path`). `scripts/setup.sh:368` has the same `/v1` locally.
- file-service: `http://file-service-svc.<sellsuki|sellsuki-dev>`

Fixed in member-api MR !141 (`fix/member-api-service-url-env`): the three values
files, the two `envDefault`s removed, and `cmd/generics_server/service_urls.go`
— a boot guard that logs ERROR naming the env var and the values file for any
outbound URL that is empty or loopback while `ENVIRONMENT` is not
local/development/test. It logs rather than exits on purpose: a missing URL
breaks only the features calling that dependency, and killing the pod would take
login, profile and theme down to protect a point-balance read.

Still missing after that MR: `FILE_SERVICE_API_KEY` (a secret, so not in the
values file) — receipt upload stays broken until someone adds it.

**Production values are inferred, not verified.** Every URL in
`values-production.yml` uses the same service names as staging, so `.octoplus`
was used there too; `tsh login` to the production cluster would have killed this
session's staging access ([[reference_teleport_session_kills_devth_access]]), so
nobody confirmed it. Whoever deploys prod must check the namespace.

**Separate trap found while committing this:** member-api's `.gitignore` had a
bare `generics_server` line (meant for the built binary at the repo root). A bare
pattern matches any path component, so it ignored the whole
`cmd/generics_server/` **directory** — new files there were absent from
`git status` and skipped by `git add`, while already-tracked files in the same
directory still showed up, which is what made the rule look harmless. Anchored
to `/generics_server` in the same MR. Worth checking for this shape in other
repos.

See [[reference_oc2plus_otp_session_fails_3rdparty_consent]] ·
[[reference_messaging_otp_needs_a_message_action_row]]
