---
name: reference_oathkeeper_global_cookie_session_is_staging
description: One Oathkeeper in ns share serves BOTH dev-th and staging-th; its global cookie_session config is the STAGING Kratos — so a dev Rule that omits cookie_session config rejects every dev session. Every dev Rule sets it explicitly.
metadata:
  type: reference
---

Verified on the staging-th cluster 2026-09-30.

- Oathkeeper is a single deployment `oathkeeper` in ns `share`; maester (`--rulesConfigmapNamespace=share`) loads Rule CRs from **every** namespace. Dev mappings also route to `oathkeeper-proxy.share:4455`.
- Its global `authenticators.cookie_session` config (`cm/oathkeeper-config`) is `check_session_url: http://kratos-public.share/sessions/whoami`, cookie `sellsuki_session_staging` — **staging**.
- A Rule with `- handler: cookie_session` and no `config` therefore uses staging Kratos. That is why BOLA's staging rule (`bola-api-auth`, ns `bola`) works with no config, and why copying it to dev would reject every dev session (login loop).
- Every dev Rule of sellsuki/octoplus/patona sets it explicitly: `check_session_url: http://kratos-public.share-dev/sessions/whoami`, `only: [ory_kratos_session, sellsuki_session_dev]`, `subject_from: identity.id`, `preserve_path: true`, `force_method: GET`, `extra_from: "@this"`.
- Mutator for BOLA must be `header` only — `id_token` injects `Authorization: Bearer` and BOLA's local_jwt revocation check then 401s (`session_revoked`).

**I got this wrong first** (told SRE "dev rule = copy staging" in BOLA-329) and corrected it in comment 45426 — check the Rule's cookie_session config before trusting "mirror staging". Dev rule landed as bridge !8. See [[reference_sre_gitops_repos_and_sync_policy]].
