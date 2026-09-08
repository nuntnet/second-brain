---
name: project_fact_vocabulary_collision
description: Two shipped defaults disagree about the pilot's Customer Fact vocabulary, which is why fact-schema validation can only check the delta
metadata:
  type: project
---

The AI chat platform ships **two defaults that name different Customer Fact
keys**, and neither is marked stale:

- `lead_rule.DefaultConfig` (chat-core, `src/entity/lead_rule/default_template.go`)
  seeds every workspace a rule conditioning on `phone_captured` +
  `name_captured`.
- `customerfact.DefaultV1Schema` (ai-platform-kit-go v0.5.0) declares six keys:
  `preferred_name`, `preferred_contact_channel`, `product_interest`,
  `budget_range`, `preferred_language`, `health_notes` — none of them those two.

Consequence, verified 2026-09-08: switching on whole-config fact validation in
`lead_rule.ReplaceConfig` refuses **every save in every workspace**, including a
PatchConfig that only changes an auto-degrade silence-day count. The chat-core
route tests fail on exactly this.

So AI-213's wiring (chat-core !79) validates only the fact keys a save ADDS
(`Config.NewFactKeysAgainst`) and grandfathers pre-existing ones. That is a
mitigation, not a resolution — an invalid seeded reference stays invalid and
silently unmatchable.

**The open decision**: are `phone_captured` / `name_captured` Customer Fact
fields (→ belong in the default schema, kit MR + version bump) or a different
kind of derived capture signal (→ should not be validated against the fact
schema at all)? Note the default rule can never fire today regardless, because
`ProfileCardSource` (AI-7's values store) is unwired.

This is the concrete content behind the old "AI-16 AC8 sign-off" blocker. The
field-set-is-admin-configurable decision (2026-09-07) dissolved the *compliance*
half of that blocker but not this one. See [[project_ai150_members_read_only]]
for the same pattern of a decision dissolving one half of a stated blocker.

Related: [[reference_ccs_ai_chat_config_namespace]],
[[case_type_setting_has_no_home]].
