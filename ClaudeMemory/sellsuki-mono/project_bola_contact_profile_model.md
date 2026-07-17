---
name: project_bola_contact_profile_model
description: "BOLA contact/profile/follower data model — how customer, phone_contact, contact_profile relate + when each is created (traced from code 2026-07-17)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
---

**BOLA has 3 entities for a person (traced from bola-backend code):**
- `customer` = **follower** (LINE identity). Created by `UpsertFollower` (follower.go).
- `phone_contact` = phone-based contact. Created by import / API / PNP.
- `contact_profile` = editable metadata (email/note/tags/custom_fields), a **SINGLE ROW SHARED** between a follower and its linked phone_contact via `followers.contact_profile_id` + `phone_contacts.contact_profile_id` (migration 0072).

**When contact_profile is created:** ONLY via the 2 edit functions (`UpsertFollowerContactProfile` follower.go:390, `UpsertPhoneContactContactProfile` follower.go:1482 — the only `CreateContactProfile` callers). So:
1. Follower from webhook (follow/message) → NO profile (UpsertFollower only writes customer + links phone_contact).
2. LIFF uid-captured → NO profile (calls UpsertFollower).
3. Import / 4. API upsert → profile ONLY if the row carries email/note/tags/custom_fields (UpsertContacts → `c.Profile != nil` → UpsertPhoneContactContactProfile). Phone/name-only → no profile.

**Contact↔OA link / follower creation happens 3 ways** (all call `LinkPhoneContactToFollower`): (a) LINE user follow/message/opens-LIFF → UpsertFollower backfills the phone_contact_followers junction (follower.go:204); (b) PNP by-phone scan (pnp.go:659/718); (c) import/API contact that includes a LINE UID (follower.go:1322).

**Editing on the Follower page = DUAL WRITE** (`UpdateFollower` follower.go:535): email/note/tags/custom_fields written to BOTH the `customer` row AND the shared `contact_profile` (sync, merge semantics per BOLA-173); **phone → customer row only** (not a contact_profile field); **name is NOT editable** (= LINE display_name). Contact page instead edits phone_contact core (first/last/phone) + contact_profile.

**TECH-DEBT (root of [[project_bola_workspace_scoping_bugs]]'s BOLA-173):** email/tags/custom_fields are stored in TWO places (customer row denormalized + shared contact_profile) and synced on edit = two-sources-of-truth (migration 0072 transition). Follower detail READ prefers contact_profile with fallback to the linked phone contact's profile. Long-term fix = contact_profile single source, customer reads via join. Also: BOLA-173 = diverged profiles when the two point at different rows; fixed by symmetric converge (MR !127).

**REFACTOR EPIC (2026-07-18):** Full single-source refactor = **Epic BOLA-272** + cards BOLA-273…284 (design doc `docs/bola-contact-model-refactor.md`, has §11 merge-policy + §12 observability). Plan: contact_profile = person node, 1 person:N follower (junction 0058), phone at phone_contact only, chokepoint eager-create at UpsertCustomer/UpsertPhoneContact, expand/migrate/contract R1/R2/R3, per-tenant rollout. **CEO-review P0 finding:** `reconcileContactProfile` (follower.go:1569-1591) is gap-fill NOT merge — fills only-when-empty, tags/custom_fields do NOT union, phone-side value silently dropped, orphan left, no tx/lock. Lazy-create hides it today; eager-create (R1) makes 2-populated-profiles the norm → routine silent data loss. Must ship real merge policy (union + conflict rule + audit + tx+lock + orphan GC + workspace guard) BEFORE eager-create.
