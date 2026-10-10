---
name: project-pstack-as-workflow-benchmark
description: "User benchmarks the sellsuki /feature workflow against cursor/plugins pstack (poteto-mode); 2026-10-10 adopted verifier agent + sequence-verifiable-units; open question whether to also adopt pstack's cite-the-principle layer or wire a central manual"
metadata:
  node_type: memory
  type: project
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-10T16:57:59.519Z
---

2026-10-10 the user asked how the sellsuki workflow differs from pstack
(https://github.com/cursor/plugins/tree/main/pstack, poteto's 24 principles + 23
playbooks) and then had two pieces ported into `/feature` (commit `759674b4` on
`fix/AI-49-safe-worker-activation`, not pushed): step 6.5 sequence-verifiable-units
and step 12.5 an independent `verifier` agent (`.claude/agents/verifier.md`) whose
PASS/PASS+NOTES/FAIL in the card ledger is the only thing that counts as AC evidence.

2026-10-11 the user chose item (1): pstack's principle mechanism. Built as
`.claude/principles/` (37 leaves: 24 adapted from pstack + 13 sellsuki-native),
`.claude/rules/principles.md` (select per task, read before citing, cite with the
decision changed), `/principles <card|task>` selector, `/feature` 0.8, and a
`## Principles applied` table in `docs/cards/TEMPLATE.md`. Commit `c5c07e3b` on
`fix/AI-49-safe-worker-activation`, not pushed.

2026-10-11 (same day, "เก็บทั้งหมดที่เหมาะกับเรา"): the rest landed in one commit on
the same branch — `.claude/verify/<surface>/` feature maps for oc2plus-backoffice,
oc2plus-member, bola-console, ccs3 (seeded from source by Explore agents; every
recipe tagged `(unverified)` until `/feature-map maintain` drives it live),
`/triage`, `/correct` + `knowledge/rule-enforcement.md`, `/pause` + `/resume` +
ledger `## Resume here`, `/feature-map`, `scripts/doctor.sh`, `scripts/survivors.sh`
(member-api 14 d: 57 merged, 0 reverted, 75% unreviewed), `scripts/log-decision.sh`,
review-bot rubric, land.md CI classification, MR detail shape, central-manual table in
`knowledge/commands.md`. **"control glass" in the user's message meant Lauren Tan's
Control Glass skill (CDP + feature map), not the Control Tower** — I misread it once.
Decided: no raw CDP; Playwright/preview tools + feature map + doctor are the equivalent.

2026-10-11 later: router rule (`.claude/rules/router.md`), test users as infra (`scripts/seed-verify.sh`), member-app login verified live, portable plugin `plugins/sellsuki-workflow` (build-plugin.py; 63 generated + 5 hand-written skills by another session; Claude/Cursor/Codex/Antigravity), first `/eval-skill` run on /triage (promoted 11/12, 10/12), branch pushed with the user's OK.

Open: the first live pass of each feature map (stack was down / sandbox could not
reach ports at seeding); the user's PO session still runs `po-team` without the
principle layer except through po-lead's new FRAME step.

**Why:** the user wants the workflow teachable and shareable to the OC2Plus team
([[project-team-workflow-sharing-goal]]); pstack is the external yardstick they
reach for, so later comparisons should start from what was already adopted.

**How to apply:** when the user mentions pstack, Control Glass, feature map or
"คู่มือกลาง", start from what is adopted (commit `c5c07e3b` and the follow-up on
`fix/AI-49-safe-worker-activation`); the next useful step is driving one surface's
map live, not more porting. The user's recurring pain is "ACs never all done, sessions
stop before finishing" — the answer built for it is `/pause` + `/resume` + the 6.7
blocked-AC rule; point there before proposing anything new. Principle leaves earn a place only
when a judgment call went wrong twice and no script catches it; otherwise it is a
`ship-check` detector or a rule. Keep the verifier
model at opus per [[feedback-subagent-model-tiering]].
