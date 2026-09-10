---
name: reference_sre_test_job_slots_cannot_chain
description: "SRE frontend CI: the *_TEST_BEFORE_SCRIPT/_SCRIPT slots run as an UNQUOTED variable expansion so `&&` and quotes arrive as literal npm args; the test jobs also lack the @sellsuki registry auth the build job sets — put the steps in a committed script file"
metadata:
  node_type: memory
  type: reference
---

**Learned the hard way on `oc2plus-linecrm-frontend-member` MR !39, 2026-09-10** (two red
pipelines before green).

## 1. The test-job script slots cannot chain commands

`templates/test.template.yml` runs the slot as `$ ${UNIT_TEST_BEFORE_SCRIPT}` — an **unquoted**
expansion. The shell word-splits it but never re-parses operators or quotes, so

```yaml
UNIT_TEST_BEFORE_SCRIPT: 'npm config set x y && npm ci'     # ✗
```

fails with ``npm error `&&` is not a valid npm option``. Wrapping in `sh -c "…"` fails the same
way (the quotes are not reprocessed either). **Fix: commit a script and let the variable name
just the file** — `UNIT_TEST_BEFORE_SCRIPT: './ci/unit-test-install.sh'` (mark it `100755` via
`git update-index --chmod=+x`).

## 2. The test jobs have no `@sellsuki` registry auth — only the build job does

`.build_frontend_npm` (`templates/build.template.yml:73-75`) runs `npm config set
@sellsuki:registry …` + `_authToken` before installing. The **test** jobs get only a `~/.netrc`
holding `CI_JOB_TOKEN`, which cannot read another project's package registry:

```
npm error 404 Not Found - GET …/projects/349/packages/npm/@sellsuki/sellsuki-bbcode-parser/… - Project not found
```

So any repo turning its unit-test job on must re-do the registry setup itself. `GITLAB_PROTOCOL`,
`GITLAB_DOMAIN` and `SELLSUKI_TOKEN_NPM_REG` **are** available on ordinary MR pipelines (verified
against a passing build job on the same MR ref), so no protected-branch dance is needed.

## 3. `node:22` exists on the internal mirror (settled)

Every frontend here pinned `node:20` and the registry rejects anonymous tag queries (401), so this
was unverifiable from the CLI — the pipeline settled it: `Using Kubernetes executor with image
registry.fountain.sellsuki.com/dockerhub/library/node:22`, and both the unit-test and build jobs
passed on it.

## 4. Turning the gate on is a deletion, and proving it works needs a red pipeline

`gitlab-ci-pipeline.generic-frontend-vue-npm-th.yml:13-15` already defaults to `npm install` +
`npm run test:unit` on `node:16`; a repo showing `echo "1 + 1"` is overriding the template locally.
Delete the override, pin a modern image, and **do not enable the E2E/INTEGRATION slots** unless
`test:e2e` / `test:integration` scripts exist — the template will run scripts that do not exist and
redden the pipeline.
A green pipeline never proves a gate works (an `echo` is green too). Land a throwaway commit with
one wrong assertion, capture the failed pipeline URL, then revert — keep both commits in history as
the card's evidence.

Related: [[reference_frontend_kit_consumption_needs_pnpm]] [[project_oc2plus_member_react_migration]]
