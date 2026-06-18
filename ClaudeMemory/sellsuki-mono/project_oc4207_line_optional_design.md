---
name: project-oc4207-line-optional-design
description: "OC-4207 chose empty-string (not *string) for optional LINE UID — affects dependent cards OC-4241, OC-3339, OC-3340, OC-3341"
metadata: 
  node_type: memory
  type: project
  originSessionId: e1faee61-774b-4b85-9a19-984881102fff
---

OC-4207 "รองรับ member ที่ไม่มี LINE linkage" — implemented 2026-06-18 on branch `feat/oc-4207` in `backend/oc2plus-line-crm-service-backoffice-api`.

**Design decision**: Option A — keep `member.Identity.LineUserID` as `string` (no entity module change). Empty string `""` means "no LINE linkage". Handled at DB + repository layer via `sql.NullString`.

**Why**: Entity module `entity v1.7.3` is shared across services; changing to `*string` would require coordinated update across all consumers.

**How to apply**: When implementing OC-4241, OC-3339, OC-3340, OC-3341 — check `LineUserID == ""` to determine if a member has LINE. Do not expect a pointer type.

**What was added**:
- `migrations/001` — nullable `line_user_id/display_name/display_image` in `member_identity`
- `migrations/002` — `member_bola_follower_link` table (member ↔ BOLA follower + LINE OA)
- `migrations/003` — `member_line_snapshot` table (display_name, picture_url, follow_status)
- `MemberBolaFollowerLinkRepository` + `MemberLineSnapshotRepository` interfaces/impls
