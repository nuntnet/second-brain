---
name: project_ccs3_qa_change_notification_agreement
description: What the CCS3 e2e QA owner asked the dev side to announce in advance, and the data-testid contract that replaced their brittle selectors — from the 2026-09-10 regression debrief
metadata:
  type: project
---

From the **CCS3 Regression Debrief, 2026-09-10** (659 tests on staging, 111 failed;
26 of them in the 8 suites that QA owner analysed). Their conclusion: **25 of those
26 were deliberate app changes, not regressions** — the test side simply learned
about them by watching a run go red.

**They asked for advance notice on exactly three kinds of change**, and this is
cheap to honour:

1. **Adding or moving a column in a main table.** One column added to the User List
   shifted every index and broke 16 cases at once.
2. **Adding a permission to Company Owner's tree.** Their role-permission assertions
   use `toEqual`, which compares contents *and order*.
3. **Changing a status code or error shape a client has to catch.** A 500 → 422 move
   (PAT-2423) silently disabled a retry whose condition was pinned to 500, with the
   message text unchanged.

**The selector contract (CCS3 MR !519, 2026-09-17).** The suite had been anchoring on
Svelte's generated scope class — `ssk-header-cell.svelte-1libewv`, a hash of the
component's compiled CSS that changes on any CSS edit and then fails as "element not
found" with nothing naming the cause. The User List table now carries semantic ids:
`user-list.table`, `user-list.header.{name,email,roles}`,
`user-list.cell.{name,email,roles}`, plus `data-user-id` per cell so a test can
address one person's row instead of counting rows. **Treat these as a contract** —
renaming one breaks e2e the same way moving a column does.

Also worth knowing when reading their reports: a suite's pass rate can hide the
finding. `TS-Accept-Invite-Link` read 11/15 — flaky-looking — until the cases were
split by whether they actually press accept: **4 of 4 that did, failed**, and none of
the 11 passes ever pressed it. See
[[reference_invite_accept_burns_the_code_on_grant_failure]].

QA owns 8 of the suites; `TS-Create-Goods-Receive` and `TS-Message-Configuration`
belong to someone else and are not in their analysis.
