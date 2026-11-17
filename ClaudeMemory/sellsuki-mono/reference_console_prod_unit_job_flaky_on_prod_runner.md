---
name: reference-console-prod-unit-job-flaky-on-prod-runner
description: "ai-chat-admin-frontend production tag pipeline runs ~3,900 unit tests on the production runner pool; v1.0.4 needed 4 runs (CasePanel countdown 3:40 vs 3:41, OpsPage re-bind, a runner node loss)"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-17T17:24:31.434Z
---

2026-11-17, console hotfix v1.0.4 (one CI variable): `unit_test_production` failed 3 times, each differently — `CasePanel.test.tsx` AI-417 countdown (expected '3:41' got '3:40'), `runner_system_failure` (node "not ready", pod vanished after 35 min), `OpsPage.test.tsx` "a failed re-bind leaves the alert in place". 4th retry passed. The production runner pool shares nodes with production workloads (SRE-1694 already removed e2e from this pipeline for that reason).

**How to apply:** for a production console tag, expect retries; retry by job id, and after 3 different failures ask nunt before more retries (option: let the prod unit job not block a config-only hotfix). The two flaky tests deserve their own fix.
