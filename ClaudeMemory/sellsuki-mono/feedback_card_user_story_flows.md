---
name: feedback_card_user_story_flows
description: "Cards written as pure technical concept are not enough — must be User Story + user flows (create/edit campaign, runtime behavior) with technical scope as supporting section"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

User correction (2026-07-08, on OC-4295/OC-4297 after I wrote them as technical specs): "ทั้งสองการ์ด ตอนนี้เขียนเป็น technical concept ซึ่งไม่พอ จะต้องเขียนเป็น User Story โดยมีเนื้อหาของ user flow ตอนที่ create campaign, edit campaign ได้ด้วย แล้วก็ระบบที่ get point จะทำงานบน event ทันที เขียนจบในทั้งการ์ดนี้เลย"

**Why:** cards grounded in code research drift toward architecture documents; devs and QA also need the merchant-facing journey (which screens, which fields, what happens when they edit a live campaign) and the explicit runtime promise (award happens real-time on event, not batch, no human approval).

**How to apply:** every feature card = (1) User Story per persona (admin ตั้งค่า / end-user ได้ผล), (2) Flow sections: Flow A create (numbered steps on real screens), Flow B edit (per-status table — draft editable, published+used = restricted, e.g. ห้ามแก้ rate ย้อนหลัง), Flow C runtime (event → award within seconds, ASCII flow), then (3) 🔧 Technical Scope as a supporting section for devs (keep the file:line evidence — don't delete it, demote it). AC groups mirror the flows (A=UI, B=runtime, C=integration). Applied as the standing structure across the loyalty cluster ([[project_loyalty_point_cluster]]).
