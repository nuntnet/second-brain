---
name: feedback_prototype_review_controls_feasibility_date_ranges
description: "nunt reviewing a data-page prototype (AI-461 margin, 2026-11-20) — every control must work, say on the page which numbers the system can't produce yet, show per-customer date ranges when periods don't align, one summary + one tab row"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 58c6eca6-d6eb-446b-bf6a-a91ba8b63329
  modified: 2026-11-20T09:44:18.278Z
---

On the AI-461 margin page prototype, nunt rejected v1 ("ยังดีไม่พอ ลองคิดใหม่ให้ครบ") and reviewed v2 with four points:

1. **"Tool ในการกดดูไม่ครบถ้วน"**: every visible control has to do something in the mock. That covers range, tabs, filters, search, sort, chart bars, rows, drawer tabs, actions and export. Prove it with a scripted click-through, not a screenshot.
2. **"ทำแบบนี้ได้จริงๆหรือยัง หรือต้องแก้ไขระบบอีกเยอะ"**: a prototype that shows numbers must say which numbers the real system cannot produce yet, and which card builds them. Put that on the page (a badge per figure plus a feasibility table), not only in chat.
3. **"วันที่ของแต่ละ company เริ่มและสิ้นสุดไม่พร้อมกัน"**: billing is anniversary-based, not calendar-based. Show the chosen range explicitly. Prorate each paid cycle into the range, and show each customer's cycles (timeline plus table).
4. **"การแยกโซนแยกแท็ปยังไม่ดี"**: one summary, then one tab row. Merge zones that are the same object. For example, trial and free customers belong in the customer list, not in a separate zone.

v1 failed earlier because it laid the card's data out as tables instead of answering the reader's questions (who, why, better or worse, which plan, what next).

**Why:** nunt judges a prototype as the real product, and the finance and sales readers need numbers they can trust.

**How to apply:** for any dashboard or report prototype:
- Write the reader questions first.
- Probe the real data sources and mark the gaps on the page.
- Make every control live and run a click-through script before showing it.

Related: [[feedback-selfexplaining-ux]], [[feedback-journey-first-before-screens-on-multi-card-features]], [[feedback_console_ux_story_presets_icons_generic]].
