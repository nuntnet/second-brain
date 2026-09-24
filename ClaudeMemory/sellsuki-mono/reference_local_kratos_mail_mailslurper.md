---
name: local-kratos-mail-mailslurper
description: Local Kratos verification/recovery emails (OTP) land in Mailslurper — UI localhost:4436, JSON API localhost:4437/mail; compose comments have the ports swapped
metadata:
  type: reference
---

Local signup/verification/recovery emails from Kratos never leave the machine — they go to the `mailslurper` docker container.

- UI: http://localhost:4436 (also `mail.sellsuki.local` via Caddy)
- API: `http://localhost:4437/mail?pageNumber=1` → `mailItems[]` with `toAddresses`, `subject`, `body` (HTML; code + link `.../self-service/verification?code=NNNNNN&flow=...`)
- `docker-compose.yml` labels 4436 "SMTP" and 4437 "UI" — both wrong; 4436 is UI, 4437 is API.
- The rtk hook rewrites `curl` output into a non-JSON summary: use `rtk proxy curl ...` and `json.load(..., strict=False)`. See [[rtk-output-can-drop-lines]].

Verification UX bugs found with this (no OTP entry after signup, resend button locked by localStorage): PAT-2730.
