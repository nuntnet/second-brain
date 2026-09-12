---
name: reference_oc2plus_gender_enum_typo_in_backoffice_spec
description: "backoffice-api's v1.yaml declares the member gender enum three times and one of them says 'not_specified' — the entity and DB only ever hold 'not_specific', so a generated client offers a value the system rejects"
metadata:
  node_type: memory
  type: reference
---

Found 2026-09-12 while wiring OC-3126's profile fields in
`oc2plus-line-crm-service-member-api`.

The canonical set lives in the shared entity module
(`gitlab.sellsuki.com/sellsuki/oc2plus/line-crm/backend/entity`,
`member/member.go:38-42`):

```go
GenderNone        MemberGender = ""
GenderMale        MemberGender = "male"
GenderFemale      MemberGender = "female"
GenderNotSpecific MemberGender = "not_specific"   // <- "specific", no "ed"
```

`backend/oc2plus-line-crm-service-backoffice-api/src/interface/fiber_server/spec/v1.yaml`
declares it three times and they disagree:

| line | value | |
|---|---|---|
| 2096 | `["male","female","not_specific",""]` | correct |
| **6889** | `[male, female, not_specified]` | **typo, and drops `""`** |
| 7006 | `["male","female","not_specific",""]` | correct |

Line 6889 is in the **member response** schema, where `gender` is a required
field — so a generated client declares a value the backend never produces and
has no name for the one it does. `ParseMemberGender` returns
`ErrInvalidMemberGender` for `not_specified`.

**How to apply:**
- Never cast a wire gender string to `member.MemberGender`. Route it through
  `member.ParseMemberGender` and map the error to a 400 — a cast writes an
  unknown value into the column instead of rejecting it. That is what
  `members_v1.go` does for the member-create path (MR !111).
- When adding a gender field to any spec in this stack, copy line 2096's form,
  not 6889's.
- Fixing 6889 is a one-word change plus a regenerate, but it is a *response*
  contract, so it needs its own MR in backoffice-api — still open as of
  2026-09-12.

Related: [[reference_oc2plus_schema_lives_in_external_repo]],
[[reference_entity_lib_tenant_kinds]].
