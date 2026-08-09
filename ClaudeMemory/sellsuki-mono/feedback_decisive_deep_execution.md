---
name: feedback-decisive-deep-execution
description: "User wants sharper, more decisive work — investigate deeply and act, not option-menus"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9e057444-0352-41d3-9463-fb0dd304ba9a
  modified: 2026-08-09T05:32:38.816Z
---

User feedback (2026-06-18): "ต้องการให้ฉลาดและคมกว่านี้" — wants sharper, more incisive output.

**Why:** I had been ending most turns with a "(ก)/(ข)/(ค) เลือกข้อไหน?" menu and deferring the
decision back to the user, and giving surface-level answers that left open questions for *them* to
resolve. That's slow and not sharp.

**How to apply:**
- When a question has an answer findable in the code/data, **go find it and resolve it** — don't offer
  to find it. (e.g. "where is the pre-orderable flag?" → trace it, don't ask permission to trace.)
- Trace the real call path / domain model before answering; surface the non-obvious insight, not the
  obvious restatement. (e.g. inventory OMS reserve has NO strict-reserve — overflow always → preorder;
  oversell-allow is an upstream decision. That's the sharp point.)
- Do the work decisively, then report what was found + done crisply. Stop the long ก/ข/ค option menus.
  Ask only when it's a genuine product/architecture decision that's truly the user's to make.

**Reinforced 2026-08-08/09 — twice in one session.** After I diagnosed problems accurately and then
ended with "บอกได้เลยว่าจะให้ลุยมั้ย", the user replied only **"แก้ไขให้หมดสิ"** and
**"แก้ไขทั้งหมดให้เทสบน Local ได้"**. Pattern to kill: *finding* the blockers and *listing* them is
not the deliverable — fixing them is. When a blocker is fixable with the tools at hand (write the
missing migration, fix the wrong env value, create the missing config file), **fix it and then
report**, even when there are many of them stacked (that run had 9 + a real code bug). Reserve the
question for things I genuinely cannot decide: destructive/irreversible actions, committing to
someone else's branch, or a product call. Cost of asking first ≈ a wasted round-trip every time.
