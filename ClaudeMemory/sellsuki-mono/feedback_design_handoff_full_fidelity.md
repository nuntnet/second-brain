---
name: feedback-design-handoff-full-fidelity
description: เวลา implement design handoff — user คาดหวังทั้งเฟรม (shell/nav ด้วย) ไม่ใช่แค่โทนสี; อย่าใช้กฎ no-mock ตัดของที่รันบนข้อมูลจริงได้
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9847fdca-bb8a-4e71-acd5-7642c3aec6d9
  modified: 2026-08-15T01:54:58.378Z
---

On the AI chat admin design handoff (2026-08-14), the first pass adopted the
design's colors/components but kept the old shell (topbar + wide text sidebar)
and dropped search/team-dropdown/compact-assign as "no backend". The user
rejected it flat: "ยังไม่ถูกต้อง แล้วยังไม่เหมือนใน design เลย" — and later
"แก้ไขเรื่อง switch ให้ใช้งานได้จริงๆ" when a control existed but led nowhere.

**Why:** to the user, a design handoff means the whole frame — the shell, nav,
and every control that CAN work — not a re-skin. And a visible control with no
follow-through (a takeover switch with no way to reply) counts as "ไม่ work"
even when its API is fine.

**How to apply:** (1) implement the design's structural shell first (icon rail,
pane layout), not just tokens; (2) apply the no-mock/honesty rule only to DATA
that doesn't exist — search over loaded rows, switchers over real lists, and
consolidated controls are all real; (3) after wiring a state toggle, walk the
next user step (took over → now reply with what?) — if the follow-through is
missing, that's the actual feature to build. Related: [[ai-chat-frontend-design]]
