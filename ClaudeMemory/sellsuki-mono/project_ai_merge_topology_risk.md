---
name: project_ai_merge_topology_risk
description: "AI platform's real risk is merge topology, not code — 206 unmerged chat-core commits sit in one open MR !10, and 11 of 18 admin-frontend ports have no backend by design"
metadata: 
  node_type: memory
  type: project
  originSessionId: 369f9f55-f1f1-4ed5-92cb-b4d1e841aa8f
  modified: 2026-08-14T09:35:19.014Z
---

Reviewed 2026-08-14 across all six AI-platform repos. The code works (all four Go
services `go build ./...` clean, MVP verified live — see
[[project_ai_mvp_integration]]). **The risk is that almost none of it is merged.**

| repo | HEAD branch | vs `main` | MR state |
|---|---|---|---|
| `sellsuki-chat-core` | `feature/AI-126-case-use-case` | **+206 / -0** | `!1`–`!9` merged; **`!10` open carries the whole 206** |
| `sellsuki-ai-agent` | `feat/e3-extraction` | +21 / -0 | `!1` open |
| `ai-platform-kit-go` | `feature/AI-122-feature-flags` | +2 / -0 | `!1`–`!3` merged |
| `rag-core` | `feature/AI-46-retrieval-acl` | +28 / **-15** | `!18` open, `!15` draft |
| `ai-chat-admin-frontend` | `feature/AI-122-flags-port` | +59 / -0 | `!1`, `!2` open |
| `sellsuki-messaging-backend` | `feature/mvp-media-and-secret-durability` | +49 / -3 vs `develop` | `!13` open → `develop` |

- **chat-core `main` HAS absorbed** AI-13/14/15/17/51/117 (MRs 1–9). Everything
  after that — Sprints 2–8, ~80 cards — is the single 206-commit MR `!10`. That
  is not reviewable or bisectable; it needs splitting into stacked MRs along the
  branch boundaries that already exist.
- **`rag-core` is the only repo BEHIND its main** (15 commits): an insurance-demo
  retrieval slice (PAT-2589, MRs !16/!17) landed on `main` from another team while
  AI-46 was in flight. Merge `main` into AI-46 before touching it.
- `kratos-ui-go` in this workspace sits on an unrelated branch
  (`feature/self-register-app-launcher`) — not AI work, but it is the login
  dependency for the admin UI.

**Admin frontend is honest, not broken.** Commit `a0b5283`
("feat(honesty)") deliberately replaced mock adapters with failing ones on the
**11 ports that have no backend**: quota, adsSpend, leadSuggestion, leadNotify,
company, companyWorkspace, permissions, audit, configVersion, kbTemplate,
kbEntry. Each fails with its own `*_LOOKUP_FAILED` code naming the missing piece.
**Wired to real chat-core:** persona, conversation/inbox, takeover, lead,
assignment, evalResult, auth (Kratos), flags. `VITE_USE_MOCKS` defaults to
**off**, so the running app hits real backends — a red screen there is the truth,
not a regression. Delete a factory from `unavailableAdapters.ts` when its backend
lands; that file is the todo list.

`kbEntry` is the special case: **rag-core already serves full CRUD**, but it
authenticates a Bearer gateway-JWT while the admin holds a Kratos cookie, and
channel-gateway (which minted those tokens) is decommissioned. Needs an auth-bridge
decision, not a new endpoint.

**Jira badly understates reality:** 121 non-epic cards — 6 Done, 61 To Do, ~54 In
Review — while git commit messages reference ~81 distinct AI-xx cards as
implemented on unmerged branches. Never treat a `To Do` here as unstarted; see
[[project_ai_sprint234_autonomous_run]] and [[reference_ai_board_stale_cards]].
