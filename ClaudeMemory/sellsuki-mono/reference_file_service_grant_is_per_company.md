---
name: reference-file-service-grant-is-per-company
description: "OC2Plus receipt upload 403s on any company the seed didn't grant sellsuki.filesystem.* to — the claim never reaches the backoffice and nothing logs an error there"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f1e1e4b4-53e4-4459-99e1-ea7fc29faaa9
  modified: 2026-09-11T11:20:11.986Z
---

`member-api` uploads the receipt photo to file-service **before** it writes the
`point_claim` row (`src/use_case/point_claim.go`, `UploadPrivate` at the file
step). file-service checks `sellsuki.filesystem.*` on the **company refID**, as
the service identity — `X-User-Id: sellsuki-system`, `X-User-Kind:
sellsuki.system`.

`scripts/seed-dev.sh` granted that on `$DEV_COMPANY_ID` only. So a company you
create yourself has no grant: the submit 403s at the upload, **no row is
written**, and the OC2Plus backoffice shows nothing — no claim, no error, no
trace to search for. It reads like "the member app isn't wired to this
company", which is not what is wrong.

**How to check it in one call** (403 = missing grant, 400 = grant fine, payload
rejected):

```bash
curl -s -o /dev/null -w "%{http_code}\n" -X POST \
  "http://localhost:8087/upload/private?refID=sellsuki.company%3A<COMPANY_ID>&accessMode=accessURL" \
  -H "X-User-Id: sellsuki-system" -H "X-User-Kind: sellsuki.system" \
  -F "file=@/tmp/x.jpg;type=image/jpeg"
```

Fixed 2026-09-11: the seed now grants every company in
`oc2plus_bola_bindings`, not one variable. Assignee kind must be
`sellsuki.system` — a role assigned to `sellsuki.user:sellsuki-system` is a
different Keto subject and still 403s.

**The other half of the same screen:** `oc2plus.pointclaim.review` is missing
from the plain `Company Owner` preset, so an admin holding only that role
cannot see คำขอแต้ม even once claims exist — see
[[project_pointclaim_permission_missing_from_owner_preset]]. Check per user
with rps `CheckPermission` (the request field is `actor`, not `user`).

See also [[reference_oc2plus_local_stack_recovery_traps]],
[[reference_file_service_keto_subject_kind]].
