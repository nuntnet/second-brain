---
name: reference-space-go-has-no-gateway-route-mb-kind-forced-user
description: SG-44 spike 2026-10-08 — space-go has no API-gateway/Oathkeeper rule in any env in git (nothing strips X-User-*); management-backend sits behind the shared Oathkeeper header mutator that forces X-User-Kind=sellsuki.user and passes X-Company-Id through; a plaintext private JWK sits in ory-helm values
metadata:
  type: reference
---

Found by the SG-44 spike (config read only, no live calls — run was 07:08, Teleport unreachable):

- **space-go**: zero hits for any space-go term across every branch of `api-gateway` and the rule repos (patona,
  sellsuki, bridge, line-diy). `space-storefront` points at placeholder hosts (`api-staging.line-diy.example.com`,
  `localhost:8080` on dev). The only `/lineagency/` rule (`linediy-service-auth`) fronts `line-diy-service-svc` with
  noop authenticator/mutator — easy to confuse because space-go's `APP_NAME` is `line-diy-go`. So on the cluster
  nothing is positioned to strip or set `X-User-Id`/`X-User-Kind` for space-go.
- **management-backend**: rule `management-backend-auth` in `infra/patona-config/manifest/{development-th,staging-th}/rule.yaml:5-30`
  via `api.{dev-th,staging-th}.patona.online/management`; the shared `header` mutator
  (`infra/ory-helm/staging-th/oathkeeper-values.yaml:253-261`) sets `X-User-Id` from the session subject and
  **hardcodes `X-User-Kind: sellsuki.user`**; `X-Company-Id` is passed through untouched. Dev has no oathkeeper
  values file of its own (assumed to share staging's). A provider/company kind can therefore reach
  management-backend only from an in-cluster caller.
- ⚠️ `infra/ory-helm/{production,staging,staging-th}/oathkeeper-values.yaml` hold a private JWK in plaintext
  (staging-th :273-290). Not copied. SRE should rotate and move to a secret.
- space-go local checkout (detached `7c3c137`) calls `NewKeto` at `cmd/generics_server/helper.go:179`, contradicting
  drafts that say the permission repo is a dummy at `helper.go:135` — verify against `origin/main` before SG-45.

**How to apply:** do not release any space-go path that trusts `X-User-*` until SRE confirms the real route and a
mutator exists (blocks SG-51 release); treat `X-Company-Id` as client-controlled everywhere; the three-case forged-header
test still needs to run live ≥09:00 with a test user/company on dev. Ledger: `docs/cards/SG-44.md`. See
[[reference-oathkeeper-global-cookie-session-is-staging]], [[project-sukispace-marketplace-ais-vision]].
