---
name: project-f08-fact-schema-versus-values
description: F08's "fact source" bundles two different ports — schema (exists, wiring gap) and values (no store anywhere); only the values half is a real cross-team blocker
metadata:
  type: project
---

The completeness audit's F08 row ("fact/schema sources are nil in Lead rule
composition") and any summary of it that says "the fact store lives on the
platform side" conflate **two different ports**. The user challenged this
framing on 2026-09-07 and was right to.

`lead_rule.SetOptionalSources(profileCardSource, factSchema, consent)` in
`backend/sellsuki-chat-core`:

1. **`FactSchemaSource`** — the field *definitions*. An implementation EXISTS
   on chat-core main (AI-208's `fact_schema_repository`, with GET/PUT per
   workspace). The nil is a **wiring gap plus a fail-closed hazard**, not a
   missing owner. See [[project-ai16-field-set-admin-configurable]].
2. **`ProfileCardSource`** — the field *values* for one contact
   (`GetFacts(ctx, workspaceID, companyID, channelIdentityRef)`). **No
   implementation exists anywhere in the codebase.** AI-7 designates the CRM
   (OC2Plus, epic E6) as the store, and E6 is platform-track, excluded from
   the pilot cut — so this is the real F08 question: does the pilot get a
   minimal values store of its own, or does E7 wait for CRM?

**Decided by the user on 2026-09-07: the values store is built on CRM.** Not a
pilot-local store in chat-core, even though chat-core already owns the schema
and assembles the profile card. So `ProfileCardSource` stays nil until
OC2Plus/CRM exposes it, and any pilot AC that needs a contact's Customer Fact
VALUES read back (AI-55 AC1 "phone/name captured -> Hot" evaluated through
EvaluateContact) cannot be evidenced from chat-core alone.

Worth checking before calling AI-55 blocked: lead_data (AI-58) stores lead
field values in chat-core's own tables, so a rule driven by lead fields is a
different path from the profile-card read — confirm which one the card's AC
actually describes.

Do not describe F08 as one blocked thing again: the schema half is executable,
the values half is not.
