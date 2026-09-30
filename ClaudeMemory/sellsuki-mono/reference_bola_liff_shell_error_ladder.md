---
name: reference_bola_liff_shell_error_ladder
description: "BOLA's LIFF shell fails in a fixed order, so the message on screen already names the cause — \"profile_failed\" means LINE, login and the in-client check all passed and the LIFF app is missing the `profile` scope, not that the browser is wrong"
metadata:
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-30T11:55:59.619Z
---

**Confirmed on dev 2026-09-30:** a newly created LIFF app had no `profile` scope;
the shell showed *"Failed to load your LINE profile. Please try again"*. Ticking
`profile` in the LINE Developers Console fixed it.

## The ladder — read the message, skip the guessing

`frontend/bola-frontend/src/pages/liff/LiffShellPage.tsx` runs these in order and
stops at the first failure, so the state on screen tells you exactly how far it got:

| message | state | what it rules OUT |
|---|---|---|
| "เชื่อมต่อ LINE ไม่สำเร็จ" / "Failed to connect to LINE" | `init_failed` | `liff.init` — wrong liffId, or endpoint URL not covered by the LIFF app |
| "กรุณาเปิดหน้านี้ผ่านแอป LINE" | `open_in_client` | you are in a normal browser — **this is what a desktop test gets** |
| (silently redirects) | `liff.login()` | not logged in |
| **"ดึงข้อมูลโปรไฟล์ไม่สำเร็จ" / "Failed to load your LINE profile"** | `profile_failed` | **init, isInClient and isLoggedIn ALL passed** — so it is not the browser and not the liffId. `getProfile()` itself failed: missing `profile` scope, or the user declined consent |
| "OA นี้ยังไม่พร้อมใช้งาน" | `oa_not_linked` | |
| "ระบบยังไม่ได้ตั้งค่าปลายทาง" | `config_error` | the per-OA LIFF destination row is missing |

**The trap to avoid:** seeing a LIFF error and assuming "you opened it outside
LINE". If that were true the message would be `open_in_client`. `profile_failed`
proves the opposite — the page really is running inside LINE.

## Scopes a BOLA LIFF app needs

- `profile` — **required**, or the shell dies at `getProfile()` as above.
- `openid` — optional but wanted: BOLA-328 attaches a LINE-signed `id_token` in the
  URL fragment so OC2Plus (OC-4523) can verify identity server-side instead of
  trusting a client-supplied `line_user_id`. Code treats it as best-effort — a
  missing scope silently omits the token and falls back to OTP, so **its absence
  shows no error at all**. Tick it when creating the app; nothing will remind you.

Console has no default that turns these on, and a LIFF app created for a new OA
starts without them.

Related: [[project_oc2plus_liff_shell_is_the_line_entry]] ·
[[reference_shared_liff_id_is_pnp_not_oc2plus]] (the shared LIFF id is PNP's, not
a general OC2Plus one — each shop creates its own under the OA's provider).
