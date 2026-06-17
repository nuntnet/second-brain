---
name: Proposals belong to Deals, not Projects
description: Proposal creation and AI generation happen in Deal context (pre-sale), not Project context. Project is created AFTER proposal acceptance.
type: project
---

## Decision (2026-03-15, confirmed by user)

Proposals must be linked to **Deals**, not Projects. The correct sales flow is:

```
Lead → Deal → Proposal (AI-generated + Smart Price Engine) → Accept → Create Project
```

### Confirmed rules
1. **`deal_id` NOT NULL** — every proposal requires a Deal. Hard DB constraint.
2. **Versioning per Deal** — `(deal_id, version)` unique. V1 rejected → V2 created. All versions kept.
3. **ACCEPTED required for Project** — can't create Project from Deal without an ACCEPTED proposal. Applies to ALL projects including internal.
4. **AI auto-context** — AI reads Deal fields + parsed briefs + meeting notes + closest past proposals from KB. No manual requirements needed (optional supplement only).
5. **V2 pre-fills from V1** — new version copies features/answers/pricing from rejected version. Incremental edits, not full regeneration.
6. **Rejection reason required** — human feedback when REJECTED. Stored on proposal. AI uses it to guide V2 adjustments.
7. **Internal projects too** — no exceptions. Every project goes through Deal + accepted proposal.

### Current state (wrong)
- `proposals.project_id` FK to `projects` — requires creating a Project before writing a proposal

### Target state
- `proposals.deal_id` FK to `deals` — required, NOT NULL
- `proposals.project_id` — optional back-link, set when Project is created from Deal
- Proposal UI lives in Deal detail, not Project detail
- AI auto-assembles context from Deal (fields, briefs, meetings, KB similar proposals)

### Spec files
- Migration spec: `docs/specs/proposal-deal-migration.spec.md`
- Phase 10 (AI Meeting Intelligence): `docs/specs/ai-meeting-intelligence.spec.md`
- Phase 7.5 (Smart Price Engine): `docs/specs/smart-price-engine.spec.md`

### Full Deal-context feature chain
1. Brief Parser (10A) → auto-fill Deal fields from customer brief
2. Meeting Prep (10C) → AI-generated questions based on Deal context
3. Meeting Transcript (10B) → summarize meetings → update Deal activities
4. AI Proposal Generation (Phase 3) → reads Deal + Knowledge Base
5. Smart Price Engine (7.5) → AI price suggestion + competitor benchmark
6. Proposal ACCEPTED → Create Project from Deal (auto-links proposals)
