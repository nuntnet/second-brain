---
name: feedback_oc_pat_board_ownership_rule
description: Board ownership rule for OC2Plus-scope work — receiving side (OC2Plus serves API / consumes events) = OC board; requesting/producing side (Patona calls/publishes) = PAT board
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

User decision (2026-07-04) on which Jira board owns OC2Plus-scope integration work, after discovering the same roadmap features carded on BOTH boards (PAT-2517/2522/2477 by another session vs OC-4321/4322/4323 by this one):

**Rule: "งาน OC2Plus-scope อยู่ board ของ OC2Plus ถ้าเป็น API ขารับ / แต่ถ้าเป็นขา request ก็ควรอยู่ที่ Patona"**
- **ขารับ (receiving)** — OC2Plus is the API provider or the consumer of events INTO OC2Plus (identity resolution, profile API served by OC2Plus, activity-ingest consumers) → **OC board**
- **ขา request/produce (requesting)** — Patona is the caller or event producer (OMS publishes order events e.g. PAT-2300 Kafka v2, Patona calls the profile API) → **PAT board**

**Why:** each team's board should hold the work its own services implement; a cross-service feature splits into two cards (producer on PAT, consumer on OC) linked together — never one card duplicated on both boards.

**How to apply:** before creating any OC2Plus×Patona integration card, decide which side implements it (who writes the code) and place the card on that team's board; cross-link the counterpart. When duplicates are found, canonical = the board matching this rule; the other gets comment + relates link + closed. Watch for parallel sessions carding the same roadmap items on the other board (happened 2026-07-03/04 — see [[project_oc_bola_domain_boundary]], control-tower data.js is where PAT-side plans surface).
