---
name: project_oc4511_4514_ux_cluster
description: "OC-4511..4514 (customer-app links/QR, member on claims, CCS3↔OC2Plus switcher, BOLA standard menu) — implemented 2026-09-09, all MRs open to develop; contracts, config, and gaps"
metadata: 
  node_type: memory
  type: project
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-09T16:08:38.234Z
---

**4 UX cards opened + implemented 2026-09-09 (all MRs → develop, none merged yet):**

- **OC-4511 "หน้าจอสมาชิก"** (user renamed from ช่องทางลูกค้า): backoffice-api !551 `GET /company/{id}/customer-app` (perm `oc2plus.member.view`; slug only from `active` BOLA binding; pages register/login/my-claims/submit-claim; LIFF URL `https://liff.line.me/{liffId}{path}`) · backoffice FE !575 `/member-screens` (copy, QR via `qrcode`). **Config not yet set in `deployment/values-*.yml`: `CUSTOMER_APP_BASE_URL`, `CUSTOMER_APP_LIFF_ID`** → page shows config-missing badges until then.
- **OC-4512 member on claims**: backoffice-api !552 (`member` on list/detail via `LEFT JOIN member m ON member_id AND company_id`; `member_id` filter on list+summary; all point_claim columns table-qualified) · FE !576 (สมาชิก column both tabs, ผู้ส่งคำขอ in dialog, คำขอแต้ม section on `/member/:id` — approve/reject there hand off to the queue with a toast).
- **OC-4513 CCS3 switcher**: CCS3 !492 (agent) + OC2Plus FE !577 (`?company_id=` accepted on entry: guard → choose-company → auto-select only if in user's list). **Gap: `gate.onMissingScope` not enforced in CCS3 switcher.**
- **OC-4514 BOLA standard menu**: bola-backend !169 (preview/apply endpoints, migration 0129 partial unique index, `ActionTypeLIFF`→`uri` fix; PNG labels EN-only) · bola-frontend !112 · backoffice-api **!553 stacked on !551** (target = `feat/oc-4511-customer-app-channels`, GitLab retargets to develop after !551 merges): `GET /v1/system/company/{id}/customer-app` behind existing `systemTokenGuard` (`SYSTEM_TOKEN`). **BOLA config**: `OC2PLUS_BACKOFFICE_INTERNAL_BASE_URL=<backoffice-api>/v1/system`, `OC2PLUS_BACKOFFICE_INTERNAL_TOKEN=<that SYSTEM_TOKEN>`.

**2026-09-10 additions:** OC-4511 menu was invisible — placed under sidebar group `main` ("ภาพรวม") which is `expandedGroups.main=false` and was empty on develop → moved under "การจัดการสมาชิก" (a279854 on !575). Local preview of all pending FE branches = local-only branch `local/ux-preview` in backoffice FE (merges !575/!576/!577/!578) — never push it, never commit its ref into the monorepo. Console-warning fix MR !578 (DefaultEmpty global registration + Oc2PlusTable watch getter). OC-4498 (member app error kinds) = member FE !37, with 5 self-review follow-ups listed in the Jira comment (open redirect via `from`, sign-in-again = reload, goHome, load-more 403, auth/membership services untouched).

**Why:** OC-4511 endpoint is Kratos+Keto only — a service caller has no identity, hence the system twin (found by cross-checking the agent's client against the route; agent had guessed `X-System-Token`, which matched backoffice-api's existing BOLA system route).

**How to apply:** merge order !551 → !553 (auto-retarget) ; !552/!576 independent ; !577 independent ; BOLA !169/!112 need !553 deployed + env set. Pre-existing FE type errors (Redeem/Reward/CampaignList/MemberList/`router includes(to.name)`) and 3 Node-25 `localStorage` spec failures (AppSwitcher/bola mock/ContactList) are NOT from these MRs — don't chase them.

เชื่อม [[project_customer_app_program]] [[project_oc4362_claim_cluster_gaps]] [[feedback_oc2plus_merge_to_develop]] [[reference_node25_localstorage_jsdom_conflict]]
