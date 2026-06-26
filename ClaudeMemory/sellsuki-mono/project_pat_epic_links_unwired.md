---
name: project-pat-epic-links-unwired
description: PAT Jira epic↔child links are NOT wired (parent field empty); roadmap/epic-index drift; 174 orphans; full gap analysis doc
metadata: 
  node_type: memory
  type: project
  originSessionId: 06c2449b-af9f-42b9-bf19-e32fb331ac0f
---

PAT is a **team-managed Jira project** (projectTypeKey software, simplified=true) → epic↔child link = the `parent` field (not classic "Epic Link").

**Core finding (2026-06-26):** the epic→child structure in Jira is **largely NOT wired**. Work is grouped by a `[tag]` in the summary (e.g. `[OMS 2.0]`, `[SukiPay]`) + a "Parent Epic: X" line in the *description prose*, but the actual `parent` field is EMPTY. e.g. `epic-index.md` lists PAT-2253/2254/2257/2258 as children of PAT-2240, but in Jira PAT-2240 has only 2 real linked children (PAT-2356, PAT-2330).

**Consequences:**
- `docs/sellsuki-roadmap.md` + `.claude/knowledge/epic-index.md` are hand-maintained → **drift / incomplete** (roadmap covers ~9 of 75 PAT epics; misses Provider PAT-2448–2452, API/MCP PAT-2052/2120/2121/2122).
- Control Tower [[project-control-tower]] capabilities #2 (epic's missing child cards vs DoD) and #4 (epic progress rollup) are **impossible to compute correctly** until links exist.

**Numbers (snapshot 2026-06-26):** 75 PAT epics (70 To Do / 5 Done, ~26 "hot" = updated ≥ 2026-04). **174 open orphans** (issuetype not in Epic/Subtask, `parent is EMPTY`, not Done): 78 hot. Of those: 40 just need re-parenting into existing epics (31 OMS 2.0 → PAT-2238–2247/2294, 9 SukiPay → PAT-2056–2069); 44 reveal **6 missing epics to create** (Payment Verification & Order Expiry, Technical Hardening/Tech-debt, Reporting & Export, Shipmunk↔Patona, OMS→OC2Plus CDP sync, Chat-commerce Auto-Payment); ~90 untagged need manual triage.

**Re-parent Round 1 DONE (2026-06-26):** 25 OMS 2.0 orphans linked to epics via Jira API (set `parent`). **KEY: the authoritative parent signal is the `epic-PAT-####` LABEL** (+ `**Epic:** PAT-####` line in description) — NOT a "Parent Epic" phrase; tag-in-summary inference mis-assigned 8/25 (corrected against label). For remaining orphans read the `epic-PAT-*` label first. **Round 2 / Group C DONE (2026-06-26):** created 2 NEW epics — **PAT-2487** [SukiPay] Payment Verification & Order Expiry (←2285/2286/1341/2175/2147/2166; fills epic-index "Epic 1" which never had a real Jira epic) + **PAT-2488** [SukiPay] Payment Link & SDK Chat/External (←2475/2415); 2270 had `epic-PAT-2244` label → re-parented to 2244 (not new). **Rounds 3–4 DONE (2026-06-26):** triaged 90 untagged (6 re-parented to existing epics; 19 parcel V1 → PAT-2241 [PO: likely superseded by PAT-2250]; 51 stale→close-review). Created **PAT-2489** "[Tech] Technical Hardening & Tech-Debt" + linked 5 (2151/2164/2064/2287/2486). **Cumulative: 64 orphans re-parented, 3 new epics (PAT-2487 Payment Verification, PAT-2488 Payment Link/SDK, PAT-2489 Tech Hardening).** Key learning held: `epic-PAT-####` LABEL is the authoritative parent signal (not prose); team-managed → Story parent must be an Epic (can't parent under a Story like PAT-2250). **Still pending: ~21 `[Technical]`-tagged orphans → PAT-2489; PAT-2473 RAG spike + reporting/Shipmunk/OC2Plus clusters need epics; ~63 warm/stale → review-for-close.** Full per-card detail in `.claude/knowledge/pat-epic-gap-analysis.md` (437 lines).

**Code-Verification Ledger added (2026-06-26):** verified "stale To Do" orphans against codegraph — **many are Done-but-card-never-closed** (Jira status ≠ reality). ✅ verified-done-in-code: CCS3 user/role/member mgmt (1141/1431/1432/1329/1372/1373/932/1434/1234 — RoleCard/PermissionForm.svelte + CCS member/role/invitation routes), receipt/thermal printer (1272/1278), courier (1324/1437 — courier_repository+kerry). 🟡 Parcel V1 (~20) = only oms2-mock → superseded by PAT-2250; AMS v2 account (~15) = only Profile read, likely covered by Kratos settings flow. 🔴 VAT setting (1473) = hardcoded only; invite-history/linkAccount = no code. **To make Control Tower % accurate: close code-verified ones as Done in Jira (live fetch auto-corrects), OR add `codeVerifiedDone` override in data.js.** Capability #4 must cross-check Jira↔code, not trust Jira status.

**Full detail + per-card inventory:** `.claude/knowledge/pat-epic-gap-analysis.md` (point-in-time; regenerate via JQL `project = PAT AND issuetype not in (Epic, Subtask) AND parent is EMPTY AND statusCategory != Done`, paginate nextPageToken 100/page). Don't re-query Jira every time — read that doc.

Stale-epic noise: ~32 PAT epics untouched in 2026 (MVP 1.x, Akita 3.1.x, OMS 1.0–1.4, CCS 1.0/1.2, QMS, i18n 1.x, PAT-1511 old white-label superseded by 2448–2452). A roadmap should filter these out (updated ≥ this year / fixVersion).
