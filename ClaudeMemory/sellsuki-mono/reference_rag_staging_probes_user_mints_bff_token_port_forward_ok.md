---
name: reference_rag_staging_probes_user_mints_bff_token_port_forward_ok
description: "Verifying rag-core on staging from an agent session — kubectl exec is denied, port-forward works, and the user mints the BFF JWT themselves (agent never reads BFF_JWT_SECRET); console playground needs real typing (2026-10-29)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 1b6127e7-7598-4b2b-acae-1879378b1723
  modified: 2026-10-29T07:07:28.846Z
---

2026-10-29 verify-g4 sweep (AI-38/41/43/46 RAG ACs):
- `kubectl exec` is denied in auto mode; `kubectl port-forward svc/rag-core-python-svc ...` works (unsandboxed, staging-th context). Read-only `get`/`logs` also work.
- The user decided: they mint the test token in their own terminal and paste it back; the agent must NOT read `sellsuki-rag-core-secret/BFF_JWT_SECRET`. Mint recipe (HS256, iss `https://chat-core.sellsuki.com`, aud `sellsuki-rag`, actor claims employee+`kb-editor` or customer, `tenant_provider_id=sellsuki`, `tenant_company_id`, 30 min) is in `docs/cards/sweeps/2026-10-29-verify-g4.md` "Needs user". Fixture companies: A `f2bf928c-…`, B `1752273e-…` (ids in docs/cards/AI-38.md).
- Console KB playground: setting the textarea via native setter does NOT enable submit; click the field and type for real. Tab buttons are `ssk-tab-button`/`ssk-button` in shadow DOM; click the inner `button` of `shadowRoot`.

**How to apply:** for a RAG verification, ask for the three tokens first, then probe through port-forward; never write a token to a file or commit. Related: [[reference_staging_verifier_preflight_tsh_and_pane_login]], [[reference_ai_chat_console_pane_shadow_dom_inputs_need_js_setter]].
