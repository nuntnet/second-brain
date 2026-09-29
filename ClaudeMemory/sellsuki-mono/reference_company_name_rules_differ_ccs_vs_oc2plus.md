---
name: reference-company-name-rules-differ-ccs-vs-oc2plus
description: "Company name validation differs by entry point — CCS allows 2-128 any chars, OC2Plus backoffice-api (entity v1.9.7 std_name) allows only Thai/Latin/digits/.-_+ and space, 5-50 — check the strictest before proposing a name convention"
metadata:
  node_type: memory
  type: reference
  originSessionId: 60eb73c1-cb6e-4d6a-b533-ee653d0d4b7c
  modified: 2026-09-29T03:02:48.293Z
---

A company can be created through CCS (HTTP/gRPC, CCS3 UI) or through OC2Plus backoffice-api, and they validate the name differently:

- **CCS** `src/entity/company/company.go:13` — `min=2,max=128`, no character rule. CCS3 UI allows brackets.
- **OC2Plus backoffice-api** `CreateCompany` → `c.ValidateCreate()` (`src/use_case/user.go:82`) with entity `gitlab.sellsuki.com/sellsuki/oc2plus/line-crm/backend/entity v1.9.7`: `required,min=5,max=50,std_name`, where `std_name` = `^[฀-๿a-zA-Z0-9.\-_+ ]+$`. The OC2Plus backoffice UI (`CreateCompany.vue:64`) matches.

**Why:** on 2026-09-28 I proposed the test-data prefix `[AUTOTEST] ` for PAT-2743 citing only the CCS 128 limit; the PO approved it, and refinement found every OC2Plus suite would get a 400. The PO then chose `AUTOTEST ` on 2026-09-29.

**How to apply:** any naming convention, prefix or length rule for companies must pass the strictest validator, which is OC2Plus (50 chars, no brackets or other punctuation beyond `.-_+`). Also remember name is mutable (PUT /v1/company/:id, gRPC InternalUpdateCompany), so a name is a weak guard on its own. See [[project-app-activation-per-company]].
