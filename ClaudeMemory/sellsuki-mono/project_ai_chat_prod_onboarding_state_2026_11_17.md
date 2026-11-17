---
name: project-ai-chat-prod-onboarding-state-2026-11-17
description: "Production AI-chat onboarding state 11-17 — CCS prod v1.7.0 far behind, Owner chat perms granted by hand in rps (389 roles), suspended re-grant CronJob, OC2Plus route abandoned for AI-466, trial-only plan decisions"
metadata:
  node_type: memory
  type: project
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-17T17:24:21.027Z
---

State of new-customer onboarding on production, 2026-11-17 (decisions by nunt):
- Production CCS is `ccs-origin-v1.7.0`, ~1,400 commits behind main: no `POST /company/{id}/chat-workspace`, no AI-289 Owner chat preset. nunt holds the CCS/rps release back.
- Only 4 of 389 Company Owner roles had `sellsuki.chat.workspace.manage`. Granted by hand with rps `/app/backfill-role-permission` Jobs (manage, read, chat_workspace.usage.read → 389 roles, recorded in docs/release/environments.yaml). CronJob `sellsuki/ai289-owner-chat-permission-regrant` (*/10) re-runs it; **nunt suspended it** — new companies need a manual `kubectl create job --from=cronjob/...` run.
- chat-core v1.1.1 (AI-289 fix) requires manage/provision to create a workspace → a new Owner without the grant gets 403.
- AI Chat no longer creates companies through OC2Plus (it forced OC2Plus's DPA, prod OC2Plus is a Feb build without the return-to-chat code, and creation did not complete there). AI-466 builds create-company inside the console via CCS `CreateUserCompany` (needs no provider permission). Provider `patona` until PAT-1529.
- Plans: trial-only first (AI-467 backend, AI-468 console): 14 days, 100 AI messages, once per company; trial end = AI stops, human inbox keeps working. Payment (checkout/return/webhook/tax invoice) is a later card set; nothing in the workspace sells a subscription today.

**Why:** prod repeatedly surprised us because staging verification ran against software production doesn't run. **How to apply:** before any onboarding work, re-check these facts on prod (read-only) — they change when CCS ships or PAT-1529 runs. Related: [[reference-providers-patona-poshmedica-company-pickers]].
