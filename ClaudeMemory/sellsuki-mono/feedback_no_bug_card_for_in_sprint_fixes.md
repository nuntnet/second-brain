---
name: feedback_no_bug_card_for_in_sprint_fixes
description: "User: don't open a Jira bug card for a defect found in work that is already in the sprint — fix it under the existing card (here OC-4523) and record it in the ledger"
metadata:
  type: feedback
---

Said 2026-09-30 ("ไม่ต้องเปิดการ์ด bug เพราะเป็นงานใน sprint") after I proposed a new Bug card linked to OC-4523 for the LINE-login defect.

**Why:** the defect was in delivered in-sprint work; a separate card adds ceremony without changing who fixes it.
**How to apply:** for a defect in something the sprint already owns, fix it, name the existing card in commit/MR text, and log it in that card's ledger; only propose a new card when the work is genuinely new scope (shipping.md §7). Ask which existing card if it is not obvious.

Related: [[feedback_no_scope_change_in_sprint]], [[feedback_no_tiny_cards_bundle_as_ac]].
