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

**Status 2026-09-10 (after PO go-ahead):** OC-4511 side DONE — backoffice-api !551 (238fcd7) drops `CUSTOMER_APP_LIFF_ID`/`liff_id`/`liff_url`, !553 merged forward (8937e47); FE !575 (bc7c083) = web links + QR + "ทางเข้าจาก LINE" block deep-linking to BOLA `/rich-menus?workspace_id=` (via `useBolaContactStore().workspaceId`); unbound CTA now → BOLA `/line-oa`, NOT legacy `/integration/line` (that screen = old LINE Modular/Messaging API, unrelated to BOLA; PO asked whether to hide it). BOLA rework DONE: bola-backend !169 @5443609 (`oc2PlusShellURL`, 3 destinations = web URLs keyed by button flow, reasons + `no_member_app_url`, `existing_rich_menu_id`), bola-frontend !112 @ca5fe55 (copy per reason, disabled toolbar w/o OA, update-vs-create label, OA-detail entry). Whole cluster verified only on local + unit tests; nothing merged — CI mirror still down.

**Member-app gap (needs a card, not a quick fix):** member-api has only `/auth/login` (legacy LINE access token via integration id) and `/company/{slug}/auth/login` (phone+OTP); no verified-LINE-identity login. The shell passes an UNVERIFIED `line_user_id` query → cannot be used to log in. So a member tapping "แจ้งใบเสร็จ"/"คำขอของฉัน" from the rich menu without a session gets OTP login once (redirect_uri preserves the handoff params), then direct. Making it seamless = shell passes LIFF id_token + member-api `POST /company/{slug}/auth/line` verifying it.

**2026-09-10 later:** legacy `/integration/line` menu hidden (backoffice FE !580, route kept until prod `integration` rows confirmed unused). Card **OC-4523** written for verified-LINE-identity login (`POST /company/{slug}/auth/line` with LIFF id_token; shell must pass `id_token`) under OC-4344, relates OC-4514.

**How to apply:** before touching LINE entry links in any OC2Plus card, route through BOLA's shell/destination model; never mint a second LIFF for the member app. Rework of !169/!112/!551/!553/!575 is pushed (2026-09-10); remaining seam = OC-4523 (verified LINE login).

เชื่อม [[project_oc4511_4514_ux_cluster]] [[project_oc_bola_domain_boundary]] [[reference_oc2plus_member_frontend]] [[project_oc4207_line_optional_design]]

**Local preview of the whole shell chain (2026-09-10):** backoffice-api, backoffice FE, bola-backend, bola-frontend each have a local-only `local/ux-preview` branch (never push/commit refs). Local env added: backoffice-api `SYSTEM_TOKEN` (random, local) = bola-backend `OC2PLUS_BACKOFFICE_INTERNAL_TOKEN`, `OC2PLUS_BACKOFFICE_INTERNAL_BASE_URL=http://localhost:8089/v1/system` (system route answered 200). Local bola DB: workspace `00000000-0000-0000-0000-000000000001` (Default) `company_id` changed from 'test-1' → `11111111-1111-4111-8111-111111111111` so the preset resolves the local company; its OAs (Kubota/TEST/Hi Family) already carry LIFF ids. BOLA local is Kratos-SSO: open via `https://bola.sellsuki.local/...` (localhost:5184 is not an allowed return_to; the Claude browser pane cannot reach the .local host — verify in the user's Chrome). !112 also got the preset card inside the template gallery (68619ce) + CI timeout fix (d441ec9).

**OC-4523 stack เคาะแล้ว 2026-09-10:** ทำบน **Vue** ก่อน แล้วย้ายไป React พร้อม OC-4501 (BE + shell id_token ไม่ผูกสแตก FE; รอ React = รอ frontend-kit ที่ยังไม่ publish) — link Relates OC-4523↔OC-4501 สร้างแล้ว, logic ต้องเขียนเป็น pure TS usecase คืน Result ตาม pattern OC-4494. ดู [[project_oc2plus_member_react_migration]]
