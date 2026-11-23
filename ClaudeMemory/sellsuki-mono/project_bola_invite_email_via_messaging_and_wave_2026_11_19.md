---
name: project_bola_invite_email_via_messaging_and_wave_2026_11_19
description: "BOLA invite email goes through sellsuki-messaging-backend with owner sellsuki.system:bola-service (BOLA-424), not SMTP/Kratos courier; BOLA bug batch 405-423 + 404/424/311-join ship as ONE wave, ONE release (nunt 2026-11-19)"
metadata:
  node_type: memory
  type: project
  originSessionId: 41e18883-006c-4805-a6b9-c81e971f2045
  modified: 2026-11-19T15:50:21.698Z
---

Decided with nunt on 2026-11-19 after BOLA-403 shipped (backend v1.0.34, frontend v1.0.26):

- **BOLA email = sellsuki-messaging-backend** `POST /v1/emails/send` (AWS SES, DB templates, idempotency). Not SMTP, not Kratos courier → `sellsuki-email-courier` (that path is Kratos auth mail only; its code is not in the monorepo). Owner/sender = `sellsuki.system:bola-service` (allowed owner kinds: sellsuki.user/system/provider/company, patona.system/store; no workspace kind). `ref_id` = workspace. Card BOLA-424.
  - Messaging's API cannot create Email provider configs yet (`coming_soon`) → SES credential + verified sender inserted into messaging DB per env; someone must provide it (asked in `docs/cards/BOLA-403-sre-request.md`).
- **One wave, one release** for BOLA-405..423 real bugs + BOLA-404 + BOLA-424 + BOLA-311 join page to main: card branches from main, pushed, no per-card MR; coordinator builds `feature/BOLA-bugs-wave1` per repo, one MR per repo. nunt's product calls: 409 implement raw_json/imagemap for real; 410 block send at 0 recipients; 414 All = everyone; 420 stamp last_login on Kratos session entry; 415 email import now (default: file fills only empty emails); 419 FE health card now, note on BOLA-358.
- BOLA-417 (OA vanished): code has no provider/company/workspace path that moves or deletes OAs; most likely soft-delete with a swallowed fire-and-forget audit. Needs a staging DB read nobody has done.

- **2026-11-20 change: provider = Mandrill, not SES** (nunt: "ใช้ mandrill จะไวกว่า"; SES verification SRE-1698 off the
  critical path). Rows live on dev/staging/production messaging DBs; production real send reached a Gmail inbox
  ("via mandrillapp.com" = no sellsuki.com DKIM in Mandrill yet). Mandrill reads only `aux_client_secret`.

**How to apply:** don't propose SMTP for BOLA again; don't open per-card MRs for this batch. Related: [[feedback_open_mr_per_wave_or_batch_not_per_card]].
