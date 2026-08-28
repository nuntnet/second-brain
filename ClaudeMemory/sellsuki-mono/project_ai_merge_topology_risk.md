---
name: project_ai_merge_topology_risk
description: "AI platform merge topology — the 206-commit MR landed, but the two codex/ai176-* baselines must be CLOSED not merged; only 19 unique files are stranded, each split across BE+FE"
metadata: 
  node_type: memory
  type: project
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-28T02:01:53.802Z
---

Superseded the 2026-08-14 snapshot (which said chat-core `!10` was open carrying 206
commits). **That landed.** Re-verified 2026-08-25.

## Where the lines actually are

chat-core `main` has absorbed the whole implementation line: tier1/intent, case,
checkpoint, playbook, sla_ladder, rolling summary, lead_reminder, sheet_export,
assignment — 68 migrations. ai-agent, ai-platform-kit-go and the admin frontend all
merged too. `fix/openapi-gate-merge-base-guard` was MR !19 and is merged — a dead branch.

Still open: rag-core `!18` (AI-46, 28 ahead / 15 behind — merge main in first),
rag-core `!15` (draft), messaging `!14`, and the four `codex/ai176-*-baseline` MRs.

## The codex baselines must be CLOSED, not merged ⚠️

`codex/ai176-chat-core-baseline` (chat-core !17) is **228 ahead / 261 behind main**.
Merging it **deletes 16,840 lines across 91 files**; the FE counterpart
(`codex/ai176-admin-baseline`, FE !3) deletes 53 files including the entire SLA-ladder
console. Between them they hold only **19 genuinely unique files**.

Those 19 files are three features, and **each feature is split across BOTH branches** —
neither half ships alone, so size the cards as BE+FE pairs:

| feature | card | FE | BE |
|---|---|---|---|
| Facebook connection BFF | AI-125 | 7 | 5 |
| Workspace discovery `GET /v1/me/workspaces` | AI-137 | 2 (adapter) | 1 endpoint |
| Kratos sessionauth middleware | AI-136 | — | 3 |

The BE files are findable with
`comm -23 <(git ls-tree -r --name-only origin/codex/ai176-chat-core-baseline) <(git ls-tree -r --name-only origin/main)`.

**AI-176's pinned baseline table is invalid** — it pins chat-core at `8a85aed` (on !17),
now 261 behind main. Corrected evidence is posted as a comment on AI-176.

## Board vs code — the board lies in both directions

- `GET /v1/me/workspaces`, Kratos `whoami` verification, and any `facebook*` file are
  **absent from chat-core main** (grep returns zero), though AI-136/AI-137/AI-125 read
  as In Review / In Progress.
- **AI-40 (Anti-Abuse) has no code anywhere** — not on main, not on !17. No blocklist,
  loop detection, or conversation budget. The 4 `RateLimit` hits on main are the circuit
  breaker and eval harness, not this feature. Same for AI-128, AI-153, AI-154.
- The reverse: **AI-132 is done** (`src/use_case/analytics.go` names AI-95/AI-132 in its
  own doc comment), as are AI-126/AI-127 and the AI-160–172 SLA-ladder set.

## Traps that own no card

- main still wires `canManage` to a **fixture array**. The day a real workspace list
  lands, the persona editor silently goes read-only for every workspace. AI-12/AI-17
  territory, tracked only as a note on FE !3.
- AI-183 needed rps to provision a role id for `chat_workspace.operator` and nothing
  owned that step — now **AI-185**.

## Outcome measured 2026-08-28 — AI-176's whole baseline table is dead

AI-176 pinned the **head SHA of five open MRs** as its deploy baseline (2026-08-15).
Verified with `git merge-base --is-ancestor` after a fresh fetch: **not one of the four
service SHAs is on its repo's mainline.** messaging-backend `328d82ed` (not on main *or*
develop), chat-core `8a85aed7` (main is **290 commits** past it), ai-agent `b47512eb`,
Admin `3f99771b` (its MR !3 is *closed*). The work landed via different branches; the
pinned MRs were closed. Pinning an MR head as a release baseline is the mechanism —
it rots silently because nothing re-checks it. Pin **main HEAD at deploy-decision time**
instead, and record real image digests after the deploy.

Also found the same day: chat-core `main` still 503s `GET /v1/me/workspaces` (unprefixed
identity kind → rps rejects). That endpoint is the *only* source the admin's
`grantedWorkspaceAdapter` reads, so a staging deploy without the fix reproduces AI-138 —
login works, every workspace-scoped page is unreachable. Fix = chat-core **!31**, green
and ready, unmerged. Must land before any admin staging deploy. See
[[rps-identity-kind-must-be-prefixed]].

Related: [[ai-board-stale-cards]], [[ai-sprint234-autonomous-run]], [[ai-mvp-integration]],
[[jira-sprint-ids-not-contiguous]], [[messaging-backend-shared-repo-traps]]
