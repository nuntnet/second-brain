---
name: project-system-testing-direction-2026-10-11
description: "2026-10-11 user approved the four-point system-testing fix: cross-service loyalty invariants Robot suite as a local gate, named scenario seeds (scripts/seed-scenario.sh), permission release-record ratchet in ship-check, doctor expected-procs + local-stack dropped-branch state; verifier step mandatory before MR"
metadata:
  node_type: memory
  type: project
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-11T08:44:17.233Z
---

Decided 2026-10-11 after OC-4734/OC-4735 showed that every tool in the workspace
was blind to defects *between* services sharing one DB (`oc2plus_crm`): three
services green, the product wrong. The user asked "what else must change in the
stack / how to test the system" and then said "แก้ไขเลย" to all four points:

1. **Cross-service invariants suite** in `testing/oc2plus-line-crm-automate-testing`
   (`TS-4-Loyalty-Invariants.robot`, tags `local_gate` / `known_defect_OC-4735`),
   run against the local stack, to become a `make loyalty-invariants` gate for
   repos touching `member_point` / `company_consent`. Tests derive from OC-4735's
   ACs, so some are red on purpose until that card ships.
2. **Scenario seeds**: `scripts/seed-scenario.sh` (expiring-lots, missing-consent-doc,
   two-companies, member-session) — idempotent, fixed id prefixes, local only.
   Feature maps and suites refer to scenario names, never hand SQL.
3. **Permission release-record ratchet** in `scripts/ship-check.py` with
   `docs/release/permission-baseline.txt` (shrink-only); `oc2plus.consent.manage`
   had shipped (OC-4089) with no row in `environments.yaml` and no local seed.
4. **Stack visibility**: `scripts/doctor.sh` expected-proc set + `scripts/dev-ensure.sh`
   restarts missing overmind instances through their own sockets;
   `scripts/local-stack.sh` records merged/dropped per branch (rebuild had
   silently dropped `feat/OC-3559-tier-ux-writing` from the backoffice FE stack).
   Rule added to `repos.md` git flow §7: Makefile/CI changes land on `main` first.

Plus: `/feature` 12.5 verifier is mandatory before any MR (the author's own browser
run is step 12, not 12.5 — that is exactly what happened on OC-4734).

**How to apply:** when a card touches shared CRM tables, run the invariants gate
locally before the MR and cite the scenario names in the ledger; when verification
needs a situation that is not a scenario yet, add it to seed-scenario.sh first.
Related: [[project-test-users-as-infrastructure]], [[project-pstack-as-workflow-benchmark]].
