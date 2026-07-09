---
name: feedback_no_scope_change_in_sprint
description: Never modify scope of a card that is In Progress in an active sprint — enhancements go to a NEW card blocked-by the in-sprint one; revert if already touched
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

User correction (2026-07-09, after I merged OC-4340+OC-3335 into OC-4245 which was In Progress with an open MR): "4245 ตอนนี้อยู่ที่ใน sprint in progress เราไม่ควรไปแก้ไข เรา revert 4245 ได้มั้ย"

**Why:** an in-sprint In Progress card is a commitment the dev is actively building against (MR open, AC agreed). Expanding its description mid-sprint silently changes the definition of done under the dev's feet and corrupts sprint metrics.

**How to apply:** before editing any card, check its status + sprint state. If In Progress (or committed in the active sprint): do NOT expand scope — put enhancements/merged content into a NEW card with `blocked-by` the in-sprint card ("เริ่มหลัง Phase 1 ship"). Consolidating old cards INTO an in-sprint card is equally forbidden — consolidate into the follow-up card instead. If already touched, revert the description verbatim + leave a comment telling the dev nothing changed for them. (Executed: OC-4245 reverted, OC-4340 reopened as the enhance card absorbing OC-3335.) Complements [[feedback_card_user_story_flows]].
