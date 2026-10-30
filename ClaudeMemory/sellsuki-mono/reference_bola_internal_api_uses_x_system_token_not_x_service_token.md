---
name: reference_bola_internal_api_uses_x_system_token_not_x_service_token
description: BOLA /internal/companies/* (BOLA-381) is guarded by X-System-Token (SYSTEM_ADMIN_TOKEN); chat-core's own X-Service-Token header gets 401
metadata:
  type: reference
---

BOLA-381 routes (`/internal/companies/{company_id}/line-groups`, `.../line-groups/{id}/auto-push-message`, `.../auto-push-messages/{id}`) authenticate with `X-System-Token` = BOLA's SYSTEM_ADMIN_TOKEN, held on chat-core as BOLA_SERVICE_TOKEN. Sending authctx's `X-Service-Token` is a 401. A disabled APM is not reused by create-or-reuse (a new id is minted), so store the latest id and re-verify. Contract: docs/cards/BOLA-381.md.
