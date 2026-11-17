---
name: rbac-preset-backfill-cards-pat2872-2873
description: "RBAC no-more-backfill plan lives in PAT epic PAT-2874 (11 child cards, moved from AI board 2026-11-17); reconciler is on rps main but CCS develop only"
metadata:
  node_type: memory
  type: project
  originSessionId: a0c4bf2e-bdf1-481b-bcff-5c37129e1d40
  modified: 2026-11-17T14:53:15.581Z
---

2026-11-17 nunt wanted the whole "new permission/role without migrate/backfill" effort on the **PAT board** so the PAT team deploys it (cards left on the AI board get no deploy). Jira MCP cannot move issues across projects, so every AI card was recreated in PAT and the AI originals got a pointer comment (status not changed, cannot delete).

**Map (epic PAT-2874):** PAT-2875 read CheckPermission/Keto direct callers (new, Phase 0) · PAT-2880 survey silent 403s (was AI-253) · PAT-2876 cache clear (AI-249) · PAT-2877 assign at new tenant rewrites old tenants (AI-250) · PAT-2878 change member role atomically (AI-251) · PAT-2879 reconciler to main + deploy (AI-252) · PAT-2881 decision preset vs custom role (AI-254) · PAT-2882 batch Keto sync (AI-256) · PAT-2883 CCS stops copying (AI-255) · PAT-2884 merge copies (AI-257) · PAT-2885 retire TUI (AI-258) · PAT-2872 guardrails + runbook (Phase 2, mine). Epic AI-248 → PAT-2874. PAT-2873 Rejected.

**Existing PAT cards kept as owners (do not duplicate):** PAT-2742 (align develop/main 7 repos; AC-09/AC-10 reconciler), PAT-1527 (Provider Owner perms, not started in code), PAT-2747 (audience guard).

**Facts checked 2026-11-17 (re-verify before relying):** rps `ReconcileRolePresets` (use case + RPC + tests) IS on origin/main; CCS `cmd/reconcile_role_presets` is on develop only. Divergence CCS main-only 53 / develop-only 162, rps 23 / 66. rps develop has two migrations numbered 0024. Reconciler covers 11 company roles by name (not Provider Owner). Not checked: rps pods on staging/prod, rps reconcile filter code, CheckPermission code, other direct Keto callers.

**MR check 2026-11-17 (git reachability):** AI-249/PAT-2876 (rps !139 develop, !141 main), AI-250/PAT-2877 (rps !140 develop + main twin) and AI-251/PAT-2878 (CCS !352 develop + main twin f0e0040f) are ALL merged to main and develop; not verified deployed (no pod check). AI-249 is insurance only: rps permission cache was never enabled on any env. AI-253/PAT-2880 has no MR, ledger docs/cards/AI-253.md only on claude/* branches, data dated 2026-09-26, 24 BROKEN row marks (e.g. N1 chat-core→file-service upload 403, member-api→file-service). CCS main resolves role by tenant-owned list (listTenantOwnedRoles), so the AI-271 cross-company lookup bug is not on main.

**Artifact** (private, nunt shares): https://claude.ai/artifact/LVjcCLiDbib6GEBDD54jgd — rule from nunt: no change-log text in it (what was cancelled/replaced is between nunt and me), only what the team needs.

**Jira MCP tips learned:** createJiraIssue `parent` accepts epic key at create; editJiraIssue `parent` must be `{"key": ...}`; PAT "Rejected" status sits in the In Progress category; writes need 10 s gaps.

See [[feedback_prior_art_sweep_every_jira_project_not_just_target]].
