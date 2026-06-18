---
name: feedback-decisive-deep-execution
description: "User wants sharper, more decisive work — investigate deeply and act, not option-menus"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9e057444-0352-41d3-9463-fb0dd304ba9a
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
