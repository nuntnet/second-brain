---
name: project_oc_bola_domain_boundary
description: "OC2Plus×BOLA domain boundary for member-follower (OC-4200) — LIFF register = 2-layer split; membership=OC2Plus, LINE/LIFF=BOLA, clean contract"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

Domain-boundary design for the [Bola x OC2Plus] Member-Follower integration (epic OC-4200). Decided 2026-06-23 after finding the epic carried **two conflicting design generations**, both wrong on domain ownership.

**Core principle:** Membership = OC2Plus domain (Bola must NOT own member-write logic). LINE / LIFF / OA / follower = BOLA domain (OC2Plus must NOT hold LINE logic). Registration via LINE LIFF crosses both → clean contract, neither side absorbs the other's logic.

**Key insight — the LIFF register page is NOT one domain's asset; it's a composition of 2 capabilities:**
1. LINE-side shell (liff.init/getProfile, capture `line_user_id`, OA/follower context) = **BOLA**
2. Membership form (phone + OTP, create member, CRM brand theme) = **OC2Plus**, called server-to-server. OC2Plus membership API is **LINE-agnostic** (receives `line_user_id` as a plain string, never touches LINE SDK).

**Both old generations were wrong:** OC-4245/4246 (new gen) made OC2Plus own the whole LIFF → OC2Plus absorbs LINE logic ❌. OC-4215/4216 (old gen) made Bola receive the form + create the member → Bola absorbs membership ❌.

**Contract:** Bola shell → OC2Plus `POST /members` (LINE-agnostic create+OTP) → OC2Plus calls back Bola `POST /contacts/link {line_oa_id, line_user_id, external_system:"oc2plus", external_id:member_id}`. Bola→OC2Plus follower events via Kafka `bola.follower-events` → OC2Plus updates `member_line_snapshot` (read-model, not source of truth). NO "Bola pushes create-member" webhook (that reverses the correct direction).

**Data ownership:** `member`, `member_identity`, `member_bola_follower_link` = OC2Plus (link table holds `bola_follower_id`+`line_oa_id` as references). `customers`(follower), LINE identity, channel token, LIFF ID = BOLA (stores `oc2plus_id` reference back). See [[project_oc4207_line_optional_design]] (LineUserID=="" = no LINE; link-table not column on member).

**Domain leaks found in code (flag):** OC2Plus currently holds LINE messaging/rich-menu (`backend/oc2plus-line-crm-service-backoffice-api` `src/use_case/line.go`, LINE_MODULE credentials) — should move to BOLA. Bola `SubmitRegistrationForm` writes registration state onto follower record — membership semantics leaking into Bola.

**Executed in Jira (2026-06-23, user confirmed).** New cards: OC-4289 (OC2Plus Membership API POST /members, LINE-agnostic), OC-4288 (tech-debt epic: extract LINE messaging OC2Plus→BOLA), BOLA-263 (LIFF register shell), BOLA-264 (BOLA POST /contacts/link + m2m machine-auth + idempotent — blocks OC-4289), BOLA-265 (BOLA Kafka producer `bola.follower-events` freeze topic+schema — blocks OC-4209). Rewrites: OC-4245 (membership form page, no LINE SDK), OC-4246, OC-4209 (keep Flow A snapshot, drop Flow B/phone-match), OC-4215, OC-4242 (absorbed OC-4205). **Consolidation (2026-06-23):** trimming OC-3339/3340 to "business-rule layer" cards created doc-only cards with no shippable artifact → dev confusion. Fixed by merging: OC-4245 now folds OC-3339 (register-new) + OC-3340 Flow B (enrich-existing) = single FE deliverable; Flow A (admin manual link, backoffice) carved into NEW card OC-4290; OC-3339 + OC-3340 CLOSED (Done, folded). Lesson: slice cards by shippable deliverable (1 card = 1 PR), not by document-layer (spec vs UI). Closed: OC-4204 (superseded by OC-4267), OC-4205 (→OC-4242), OC-4216 (reversed contract). Keep-aligned: OC-4207, OC-4241, OC-4263, OC-3341, OC-4258, OC-4206, OC-4267.

**follow_status canonical vocab = `followed`/`unfollowed`/`unknown`** — FROZEN in OC-4207 (member_line_snapshot owns it). Code reality (bola-backend customer.go:8): producer currently emits `following`/`blocked` → BOLA-265 maps following→followed, blocked→unfollowed. OC-4209 + BOLA-263 conform-notes appended.

**Capability-driven FE (OC-4242, decided 2026-06-23):** contact list is dynamic-by-domain — backend (OC-4241) returns `capabilities.bola = none|pending|active` (from QMS plan + binding); FE composes 2 modules (membership core shell always-on + BOLA overlay mounted only when active). No-plan → Upsell CTA (disabled filter + "activate BOLA plan" link). Backend must NOT return follower/linked rows when none/pending (enforce, not FE-filter). BOLA-264 = token-per-workspace m2m (new `workspace_service_tokens` SHA-256 + `WorkspaceServiceTokenGuard`, separate from global SystemAdminGuard); OC-4263 frontend = provider-management-frontend (Svelte), adds service-credential mgmt (show-once/rotate/revoke).

**Code facts found (must fix before build):** (1) OC-4241 has NO code yet (To Do) — hard blocker for OC-4242. (2) enum bug: code emits `LineFollowStatusFollowing="following"` — conflicts canonical `followed` (OC-4207); fix before OC-4241 reads it. (3) binding_status enum mismatch: OC-4267 spec says `failed` but code has `pending/active/revoked` (no failed) — align. (4) provider-management-frontend lacks i18n infra + secret-display pattern. (5) OC-4263 too big → slice A(QMS gate+list)/B(binding badge)/C(credential); CCS proxy backend card for generate/rotate/revoke missing.

**Still open:** BOLA-264 3 decisions (header name Bearer vs X-Service-Token sync with OC-4289; token-delivery CCS-push vs OC2Plus-pull; AC6 revoked→403). OC-4289 priority bumped to High (done). OC-4215 deferred (P3). stale link OC-3340→OC-4204 removed (done). BOLA cards live in Jira project BOLA (id 10126) — see [[reference_bola_jira_project]]. Jira MCP returns cross-project contaminated rows under load — see [[project_oc_epic_backlog_triage]].
