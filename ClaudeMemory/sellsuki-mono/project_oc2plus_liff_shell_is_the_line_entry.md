---
name: project_oc2plus_liff_shell_is_the_line_entry
description: "LINE entry into the OC2Plus member app goes through BOLA's LIFF shell (per-OA LIFF id, flow → DB destination → web URL + identity handoff) — NOT a member-app LIFF; OC-4511's CUSTOMER_APP_LIFF_ID and the first OC-4514 implementation contradict this and need rework (2026-09-10)"
metadata: 
  node_type: memory
  type: project
  originSessionId: f1e1e4b4-53e4-4459-99e1-ea7fc29faaa9
  modified: 2026-09-10T04:39:08.108Z
---

**The LINE-side entry model (confirmed by the user 2026-09-10: "เราใช้ liff shell destination"):**

- BOLA owns the LIFF: `lineOA.LiffID` per OA, fallback env `SHARED_LIFF_ID` (same helper the PNP greeting link uses).
- Rich-menu / link URL = the **shell**: `https://liff.line.me/{liffId}?liff_id={liffId}&line_oa_id={oa}&flow={flow}` → bola-frontend `/liff/shell` (`src/pages/liff/LiffShellPage.tsx`): `liff.init` → profile → `GET /v1/public/liff/destination?line_oa_id&flow` → `window.location.replace(destination + ?line_user_id&line_oa_id&display_name&picture_url&follow_status)`.
- Destination rows = `liff_shell_destination (line_oa_id, flow) → DestinationURL` (**plain web URL**, admin-editable in LINE OA detail; default flow `register`).
- Member app `/{slug}/register` (MembershipView) already consumes the handoff (`line_user_id` → link key, `line_oa_id` → channel). `/{slug}/point-claims` and `/point-claims/new` do **not** yet — they fall back to phone+OTP web login.

**What contradicts it (as of 2026-09-10, all unmerged):**
- OC-4511 (backoffice-api !551 / FE !575) added `CUSTOMER_APP_LIFF_ID` and builds `https://liff.line.me/{thatId}/{slug}/{page}` — a second LIFF concept; the "ระบบยังไม่ได้ตั้งค่า LIFF" badge the user saw comes from this. Decision proposed: drop the LIFF column/config, keep web URL + QR, point admins to the BOLA preset for LINE.
- OC-4514 (bola-backend !169) puts OC-4511's LiffURL on the buttons and upserts the `register` destination **to that liff.line.me URL** — should be shell URLs with `flow` = register / submit-claim / my-claims and **three** destinations pointing at the member-app **web** URLs; readiness `no_liff` should look at the OA/shared LIFF, not OC2Plus config.
- Member app must accept the shell handoff on the two claims pages too, or buttons 2–3 bounce to OTP login.

**How to apply:** before touching LINE entry links in any OC2Plus card, route through BOLA's shell/destination model; never mint a second LIFF for the member app. Rework of !169/!575/!551 awaits the user's go-ahead on the proposed direction.

เชื่อม [[project_oc4511_4514_ux_cluster]] [[project_oc_bola_domain_boundary]] [[reference_oc2plus_member_frontend]] [[project_oc4207_line_optional_design]]
