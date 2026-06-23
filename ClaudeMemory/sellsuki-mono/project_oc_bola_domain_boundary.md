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

**Card re-map (OC-4200, 18 issues):** 11 align (keep): OC-4200, OC-4207, OC-4241, OC-4242, OC-4263, OC-3341, OC-4258, OC-4206, OC-4267, OC-3339*, OC-3340* (*adjust dep off OC-4245). 7 to fix: OC-4245 split (Bola shell → BOLA project + OC2Plus membership page), OC-4246 reframe, OC-4216 close/replace, OC-4215 rewrite+reparent (its parent is OC-4223 not OC-4200), OC-4209 rewrite (keep Flow A snapshot, drop Flow B auto-link + phone-match), OC-4205 merge into OC-4242, OC-4204 close (superseded by OC-4267). 3 new: OC2Plus Membership API `POST /members`, BOLA LIFF shell, LINE-messaging extraction OC2Plus→BOLA. BOLA-side cards go in Jira project BOLA (id 10126), not OC — see [[reference_bola_jira_project]]. Nothing actioned in Jira yet (awaiting user confirm of the boundary).
