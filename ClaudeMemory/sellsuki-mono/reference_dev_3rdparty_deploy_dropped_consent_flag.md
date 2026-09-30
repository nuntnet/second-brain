---
name: reference_dev_3rdparty_deploy_dropped_consent_flag
description: "octoplus-dev 3rdparty-api lost CONSENT_ACCEPTANCE_ENABLED at revision 44 (7601dacf) although the repo file at that sha has \"true\" — member consent 'accept' returned 503 consent_acceptance_unavailable; live env list matched main's values (28 vars), cause unconfirmed"
metadata:
  type: reference
---

Consent acceptance (PUT /v1/consent-policy/acceptance) fails closed with 503 `consent_acceptance_unavailable` when the flag is unset (default false, `use_case/consent_policy_acceptance.go:34`); reads still succeed so the page renders and only the button fails.

Evidence from ReplicaSet history (read-only, `kubectl -n octoplus-dev get rs`): revs 37–42 `false`, rev 43 (73367273) `true`, rev 44 (7601dacf, 2026-09-29 13:08Z, pipeline #62014) **absent**. Rev 44 was created at the pipeline's deploy time, so it was not a hand edit. Repo `values-development.yml` at 7601dacf has 29 vars incl. the flag; the deployed manifest has 28 = exactly the variable list of `main`'s file.

Hypothesis, NOT confirmed: the dev deploy job renders values from `main` rather than the built branch (needs a read of `sre/deployment/pipeline-deployment` or the job log). If true, any env var added only on develop never reaches dev. `kubectl set env` is a temporary fix that the next deploy will overwrite.

Related: [[reference_ccs_deployment_values_live_in_the_repo]], [[project_oc4340_consent_acceptance]] (if present), [[reference_manual_staging_gate_silent_drift]].
