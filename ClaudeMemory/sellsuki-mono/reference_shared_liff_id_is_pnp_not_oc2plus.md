---
name: reference_shared_liff_id_is_pnp_not_oc2plus
description: "BOLA SHARED_LIFF_ID is the PNP/LON greeting LIFF (UID-capture page), NOT a fallback for OC2Plus menus — OC-4514 borrowed it by mistake; OC2Plus LIFF must be merchant-created per OA and saved on the LINE OA setting page"
metadata:
  node_type: memory
  type: reference
  originSessionId: 0f82b824-5c8b-4490-a5fb-553341a58b63
  modified: 2026-09-27T14:28:16.626Z
---

`SHARED_LIFF_ID` (bola-backend `cmd/bola_server/main.go:97`) was added in `c19a743` (2026-04-02) "feat(pnp): shared LIFF greeting — capture LINE UID on button click". Every definition says "LIFF app ID for PNP greeting link tokens"; its real consumer is `pnp.go:580`, which opens the "Universal LIFF UID capture page".

`5443609` (OC-4514, 2026-09-10) reused it as the fallback in `resolveShellLIFFID` / `oc2PlusLIFFSource`, and BOLA-328's oa-info (`liff.go`) does the same. A LIFF app has exactly ONE endpoint URL (set in LINE Console), so an OC2Plus menu button falling back to it lands on the PNP capture page, not `/liff/shell` (bola-frontend `App.tsx:244-251` keeps them separate). Result: OAs with no LIFF are reported "ready", and the readiness copy "เปิดผ่าน LIFF กลางของ workspace" is a false claim. Test `TestPreviewOC2PlusStandardMenu_SharedLIFFIDFallback_MakesItReady` codifies the bug. Fix carded as BOLA-333 AC-00. Not yet confirmed by tapping on dev.

**Why this matters:** on 2026-09-27 I told the user "no company needs to create a LIFF, the shared one covers it" — wrong, and it went into two Jira cards before the user caught it ("share_liff_id ชวนให้เข้าใจผิด … มาจาก lon greeting"). The name `liff_source = "workspace_shared"` makes it worse: it is neither per-workspace nor meant for OC2Plus.

**How to apply:** OC2Plus LIFF = merchant creates it in LINE Developers (LINE Login channel under the **same provider** as the OA's Messaging API channel — LINE issues user IDs per provider), names it, sets Endpoint URL = BOLA `/liff/shell`, scope `openid`+`profile`, and saves the LIFF ID on the LINE OA setting page. One LIFF serves all flows (`?flow=`) and can be reused across OAs in the same provider. Before calling any env var or config "shared/default", read the commit that introduced it — the name is not the purpose.

Related: [[project_oc2plus_liff_shell_is_the_line_entry]] · [[feedback_verify_absence_claims]]
