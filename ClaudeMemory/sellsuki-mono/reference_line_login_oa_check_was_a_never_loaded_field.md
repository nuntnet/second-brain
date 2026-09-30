---
name: reference_line_login_oa_check_was_a_never_loaded_field
description: "OC-4523 LINE id_token login returned 403 oa_not_bound for everyone because BolaBinding.LineOaID was never selected (table has no such column) — unit tests set it by hand; now decided by workspace via BOLA oa-info.workspace_id, so a company may have many OAs"
metadata:
  type: reference
---

`POST /company/{slug}/auth/line` compared `binding.LineOaID != in.LineOAID`; `GetBySlug` selected only `company_id, binding_status` and `oc2plus_bola_bindings` has no OA column, so it failed for every user in every env. Green tests set `LineOaID` in the mock (same shape as OC-4089: fixture and code share one author).

Fix (member-api !173 + bola-backend !208, merged 2026-09-30): the binding carries `bola_workspace_id`; BOLA's public `oa-info` returns `workspace_id`; the login refuses unless they match, **before** LINE verifies the token. Empty on either side is refused (older BOLA / pending binding). Consequence: a company with several OAs in its one workspace works, sharing a LIFF channel or not. Deploy order: BOLA first.

The audit reason `oa_not_bound` is shared by "no OA", "wrong workspace" and "company not open to members" — it cannot tell them apart; the Warn log line does (`bola_reported_workspace_empty`, `binding_workspace_empty`).

Unverified as of 2026-09-30: whether `sub` matches across OAs of *different* LINE providers (member would be re-sent to register), and any end-to-end tap in real LINE (dev was shut down at 21:00 before it could be confirmed).

Related: [[project_oc4523_line_login_cards]], [[reference_member_api_to_bola_link_never_worked]], [[project_line_setup_dual_surface_direction]].
