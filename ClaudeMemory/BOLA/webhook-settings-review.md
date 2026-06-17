# Webhook Settings Code Review - Feb 27, 2026

## Issues Found & Fixed

### 1. Missing Validation (CRITICAL)
**Issue**: `line_oa_id` not validated in `CreateWebhookSetting()` even though DB schema has NOT NULL
- Code allowed creating webhooks with empty line_oa_id
- Would fail with cryptic database error instead of clear validation error

**Fix**:
- Added validation check in `src/use_case/webhook_setting.go` line 69-71
- Returns `webhook.ErrSettingLineOAIDRequired` if line_oa_id is empty
- Matches database constraint and business requirements

### 2. Missing Error Type
**Issue**: No error defined for missing line_oa_id

**Fix**:
- Added `ErrSettingLineOAIDRequired` to `src/entity/webhook/errors.go`

### 3. Documentation Inaccuracy
**Issue**: Spec said line_oa_id was optional for HOOK type, contradicted DB schema (NOT NULL)

**Fix**:
- Updated `docs/specs/webhook-settings.spec.md`:
  - Changed line_oa_id from "optional" to "**yes** (required)"
  - Added validation section noting line_oa_id requirement
  - Added "Architecture: LINE OA Scope" section explaining why scoping is required
  - Clarified that HOOK webhooks also require line_oa_id for proper routing/isolation

## Architecture Principle

**All webhooks are per-LINE-OA**: Webhooks must be scoped to exactly one LINE OA because:
1. Event routing - LINE events for OA-A only trigger OA-A's webhooks
2. Security - Each OA has its own webhook URL and signature key
3. Message delivery - Webhook actions send to followers of the correct OA
4. Multi-OA isolation - Prevents cross-OA interference

This applies to both:
- **LINE-HOOK**: Auto-created per OA (obvious)
- **HOOK**: User-created for integrations (also per OA, not shared)

## Code Status
- ✅ Entity layer: Correct (WebhookSetting.LineOAID required)
- ✅ Use case layer: NOW CORRECT (validates line_oa_id)
- ✅ Repository layer: Correct (DB schema enforces NOT NULL)
- ✅ HTTP handlers: Correct (passes line_oa_id through)
- ✅ Documentation: NOW CORRECT (clarifies requirement)

## Testing Needed
When implementing/testing, verify:
1. POST /v1/webhook-settings without line_oa_id → 400 error
2. POST /v1/webhook-settings with line_oa_id → success
3. Both LINE-HOOK and HOOK require line_oa_id
