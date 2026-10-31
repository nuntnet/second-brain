---
name: ai-chat-settings-redesign-v9-approved
description: "AI chat settings redesign (case type → pipeline model) — v9 prototype approved 2026-10-31, review ACTIVE, first cards AI-357..360 cut; where to resume"
metadata:
  node_type: memory
  type: project
  originSessionId: 8b69c115-30aa-4ff4-b9c5-86fafae17c77
  modified: 2026-10-31T06:29:37.327Z
---

nunt approved prototype v9 and the case-type → pipeline domain model on 2026-10-31. Entry point for anyone resuming: `docs/design/HANDOFF-ai-chat-settings-redesign-2026-10-31.md` (on root branch `fix/AI-49-safe-worker-activation`).

State after that session:
- Design review `docs/design/ai-chat-domain-model-review-2026-10-31.md` is ACTIVE; D-079..D-089 answered (D-079 changed: type change only as close + linked Case, per D-091). **D-095 (cross-channel customer merge) is still open** although the handoff said all were answered; it only gates the "Later" roadmap row.
- Cards cut in Jira project AI: AI-357 (M0 reminders off), AI-358 (M1 case-type scoping), AI-359 (permissions: rps catalog/presets/backfill, blocked by AI-254), AI-360 (chat-core authz + /me/permissions, blocked by AI-359). Text in `docs/cards/drafts/cards-2026-10-31/`.
- Not yet cut: M2, M2b, M3, M3b, M3c, M3d, M4, M5 (must be split by prototype area first), M6, Later. Old cards AI-55/312/313/314/317/318/355 still need rewording per review §7 (Jira text of those was never read this session).

**Why:** the handoff was written because the coordinator account's tokens were running out; the next session has to know what is decided vs still open.
**How to apply:** resume with the card drafts index, not the handoff's roadmap table alone. UI is not redone in Claude Design: v9 HTML is the design; build console from it on a mock adapter in parallel with backend cards.
