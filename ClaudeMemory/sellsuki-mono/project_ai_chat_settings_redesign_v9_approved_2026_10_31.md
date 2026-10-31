---
name: ai-chat-settings-redesign-v9-approved
description: "AI chat settings redesign (case type → pipeline model) — v9 prototype approved 2026-10-31, review ACTIVE, first cards AI-357..360 cut; where to resume"
metadata:
  node_type: memory
  type: project
  originSessionId: 8b69c115-30aa-4ff4-b9c5-86fafae17c77
  modified: 2026-10-31T14:33:17.914Z
---

nunt approved prototype v9 and the case-type → pipeline domain model on 2026-10-31. Entry point for anyone resuming: `docs/design/HANDOFF-ai-chat-settings-redesign-2026-10-31.md` (on root branch `fix/AI-49-safe-worker-activation`).

State after that session:
- Design review `docs/design/ai-chat-domain-model-review-2026-10-31.md` is ACTIVE; D-079..D-089 answered (D-079 changed: type change only as close + linked Case, per D-091). **D-095 (cross-channel customer merge) is still open** although the handoff said all were answered; it only gates the "Later" roadmap row.
- Cards cut in Jira project AI: AI-357 (M0 reminders off), AI-358 (M1 case-type scoping), AI-359 (permissions: rps catalog/presets/backfill, blocked by AI-254), AI-360 (chat-core authz + /me/permissions, blocked by AI-359). Text in `docs/cards/drafts/cards-2026-10-31/`.
- Second set cut 2026-10-31: AI-361 (M2), AI-362 (M2b), AI-363 (M3), AI-364 (M4), AI-365 (M3b), AI-366 (M3c), AI-367 (M3d-1), AI-368 (M3d-2); text only in Jira, index in the same drafts README.
- Design gate cut: AI-369 (real visual design on the console DS + data-needs table; blocks AI-361/364..368 at the MR). frontend-kit (GitLab project 856) ships NO components: core+adapters+preset-sellsuki, DS stays `@sellsuki-org/sellsuki-components` (console pinned 0.27.0-beta.1, brand ccs3; kit wants ^0.27.1 and React 19, console is React 18.3). nunt's idea that the kit makes components easier to edit was NOT found in the kit's main (card O1). D-101 = build M5 on kit or current structure.
- Not yet cut: M5 (split by prototype area; v9 is a behaviour prototype with 0 `ssk-*` uses, so a real visual design pass is needed first), M6, Later. Old cards reworded 2026-10-31: AI-312, AI-313 edited in place; AI-317 title prefixed "[ถูกแทนที่ ...]" (status/sprint left for nunt); AI-55, 314, 318, 16, 355 comment-only because Sprint 21 is active.

- DS version, settled 2026-10-31: UX/UI's "0.27.1-beta.1" = npm `beta` tag = `0.27.0-beta.1` (nunt confirmed), already on the console. Public `0.27.1` is an OLDER line (trial: 14 tsc errors on `<ssk-modal>`, 2/1824 tests fail, no ssk-tab/ssk-search/ssk-clear/ssk-dropdown-listbox, no DS 1.0 font tokens) → kit preset `^0.27.1` would move the console backwards. Open: which published version has DS 1.0 AND fits `^0.27.1`. Analysis doc: `docs/design/ai-chat-console-frontend-kit-adoption-2026-10-31.md` (PROPOSAL); D-102 = mobile route (default A).

**Why:** the handoff was written because the coordinator account's tokens were running out; the next session has to know what is decided vs still open.
**How to apply:** resume with the card drafts index, not the handoff's roadmap table alone. v9 is a prototype (structure + behaviour), not a visual design: M5 needs a design pass against `ssk-*` (Claude Design is reasonable) before building; backend M0–M4 does not wait for it.
