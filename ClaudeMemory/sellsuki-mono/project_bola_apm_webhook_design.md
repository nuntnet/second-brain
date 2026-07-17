---
name: project_bola_apm_webhook_design
description: "BOLA Auto Push Message (APM) webhook design — webhook_setting overloaded/now-optional, LINE flex empty-text 400, template field substitution rules"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
---

**APM (Auto Push Message) = event-triggered push:** external system POSTs to `/webhook/apm/{apm_id}` → BOLA substitutes `{field}` placeholders from the payload → pushes to the configured target (follower/segment/all/**line_group**/LON/dynamic). Trigger URL is the APM's own `/webhook/apm/:id` (the UUID is the only mandatory secret).

**webhook_setting is OVERLOADED + was a premature abstraction (user flagged, agreed):** one `webhook` entity carries 3 meanings — LINE-platform webhook (`/webhook/:line_oa_id`), custom HOOK inbound trigger (what APM uses), and legacy outbound-forward fields. Only APM consumes the HOOK type. It only gives APM: SecurityToken + AllowedIPs + name. Fixes shipped: **APM create now auto-provisions a HOOK webhook_setting when `webhook_setting_id` is blank** (MR !135 BE / !99 FE) so users don't pre-create one; FE defaults to "Auto-create (recommended)". Follow-up backlog: rename "Webhook Settings" → "API Triggers", move token/IP onto APM, deprecate the table.

**Template substitution rules (fixed this session, MR !132/!133):**
- `{field_name}` placeholders (regex `[a-zA-Z_][a-zA-Z0-9_]*`), 3 substitution funcs: `substituteTemplateFields` (text), `substituteFields` + `walkAndSubstitute` (flex). Values are `html.EscapeString`'d; flex is injection-safe because it parse→substitute-string-leaves→re-`json.Marshal`.
- **Missing field is now OPTIONAL → substituted with "" (was a hard ErrMissingWebhookField that failed the whole delivery).**
- **LINE Flex API REJECTS text components with empty/whitespace `text` (400 "must be non-empty text").** So after substitution, any flex text node that ends up empty must become `"-"` (fixed via walkAndSubstitute); whole empty text message → "-" too. This bit twice: missing field AND explicitly-empty payload value both produce empty → 400.

**Other gotchas found:** every error from `CreateAutoPushMessage` was UNMAPPED in helper/errors.go → returned generic 500 (fixed → 4xx, MR !135). `DeliveryLog.ErrorMessage` was wired end-to-end but never SET → UI Error column always "-"; now populated on all failure paths + several early-returns left the log stuck `pending` (fixed MR !133). Related: [[project_bola_contact_profile_model]], [[project_bola_is_enabled_int_bool_mismatch]].
