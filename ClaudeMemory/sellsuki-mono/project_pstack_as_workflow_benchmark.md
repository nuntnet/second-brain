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

Still open: (2) a real central manual — Control Tower README §9 and
`.claude/knowledge/commands.md` do not mention `/feature`, `shipping.md`,
`verifier` or `/principles`.

**Why:** the user wants the workflow teachable and shareable to the OC2Plus team
([[project-team-workflow-sharing-goal]]); pstack is the external yardstick they
reach for, so later comparisons should start from what was already adopted.

**How to apply:** when the user mentions pstack or "คู่มือกลาง", do not re-port what
is adopted; pick up from the open item above. Principle leaves earn a place only
when a judgment call went wrong twice and no script catches it; otherwise it is a
`ship-check` detector or a rule. Keep the verifier
model at opus per [[feedback-subagent-model-tiering]].
