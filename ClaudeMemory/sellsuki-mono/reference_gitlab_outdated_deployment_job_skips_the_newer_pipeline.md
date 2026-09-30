---
name: reference_gitlab_outdated_deployment_job_skips_the_newer_pipeline
description: When two develop pipelines overlap on a congested runner, the NEWER pipeline's integration/e2e can fail with failed_outdated_deployment_job and its build+deploy get skipped while the OLDER pipeline deploys the older sha — retry the failed jobs by id
metadata:
  type: reference
---

Seen on bola-backend 2026-09-30 (pipelines 62181 sha 2feba8f7 = !202, 62192 sha 56b6a812 = !203). Runner cluster was full (`waiting_for_resource`).

- 62192 (newer, contained BOLA-330): `integration_test_…` and `e2e_test_…` → `failed` / `failed_outdated_deployment_job`, so `build_image` and `deploy_development` → `skipped`. Meanwhile 62181 (older) still built and deployed `2feba8f7`. Result: dev ran code WITHOUT the newer merge although its pipeline showed nothing red about the code.
- Fix that worked: retry only the two failed jobs by id (`glab api -X POST projects/<id>/jobs/<job>/retry`); build/deploy went back from `skipped` to `created` and later ran. Not the whole pipeline.
- Each extra merge to develop while this is happening starts another pipeline and can outdate the previous one again — hold further merges until the deploy you care about lands.
- **Verify by the pod, not by the pipeline colour:** `kubectl … get deploy -o jsonpath='{…image}'` and the env on it. A watch on "image changed" fires for the OLD pipeline's deploy too — match the exact sha.
Related: [[reference_glab_auto_merge_merges_immediately_without_ci_gate]].
