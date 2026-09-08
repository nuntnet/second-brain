---
name: project_fact_vocabulary_collision
description: phone_captured/name_captured are derived signals, not Customer Fact fields — decided 2026-09-08, and why that unblocked whole-config fact validation
metadata:
  type: project
---

**Decision (2026-09-08, by the user): `phone_captured` and `name_captured` are
DERIVED SIGNALS, not Customer Fact fields, and are never validated against a
workspace's Customer Fact Schema.**

## The collision this resolved

Two shipped defaults named different vocabularies, neither marked stale:

- `lead_rule.DefaultConfig` (chat-core, `src/entity/lead_rule/default_template.go`)
  seeds every workspace a rule conditioning on `phone_captured` + `name_captured`
- `customerfact.DefaultV1Schema` (ai-platform-kit-go v0.5.0) declares six other
  keys: `preferred_name`, `preferred_contact_channel`, `product_interest`,
  `budget_range`, `preferred_language`, `health_notes`

Consequence, measured: switching on whole-config fact validation in
`lead_rule.ReplaceConfig` refused **every save in every workspace**, including a
PatchConfig that only changed an auto-degrade silence-day count. chat-core's own
route tests fail on exactly that, which is how it surfaced.

## How it was implemented (chat-core !79)

A fourth `ConditionSource`. `Condition.Source` already documented that "the
source decides which catalog the condition is VALIDATED against", so:

- `src/entity/derived_signal/` — closed, code-owned catalog. Admins cannot add
  to it. `Is(key)`, `All()`.
- `SourceSignal` conditions are checked against that catalog (typos → 
  `UNKNOWN_SIGNAL`, 400, message lists the whole catalog) and never against the
  schema. `Rule.SignalKeys()` vs `Rule.FactKeys()`.
- `Condition.ResolvedSource()` — a **fact**-sourced condition naming a catalog
  key reads as a signal. Needed because every config seeded before the
  distinction stored them as fact conditions. Same choice `Source.Resolved()`
  already made for the pre-AC9 empty string: no column rewrite.
- `Input.Facts` stays the transport for both — governance differs, not where
  the value comes from. So a fact field colliding with a signal key costs a
  lost check, not a wrong read.

This let whole-config validation be restored: the earlier delta/grandfathering
workaround (`Config.NewFactKeysAgainst`) was **deleted**.

## Still true

Nothing WRITES these signal values yet — `ProfileCardSource` (AI-7's values
store) is unwired in main.go, AI-85 owns building it. So a rule on a signal
cannot match today, same as a rule on any fact. The catalog exists so a rule can
be *validated* now.

`DefaultPhoneCapturedFactKey` / `DefaultNameCapturedFactKey` are kept as aliases
of the catalog constants; the "FactKey" in those names is a known misnomer left
for a separate mechanical rename.

Related: [[project_ai16_field_set_admin_configurable]] dissolved the *compliance*
half of the old "AI-16 AC8 sign-off" blocker; this decision dissolved the
*content* half. [[reference_testify_permissive_default_wins]] bit again here.
