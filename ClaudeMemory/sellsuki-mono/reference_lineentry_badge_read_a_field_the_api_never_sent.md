---
name: reference_lineentry_badge_read_a_field_the_api_never_sent
description: "The backoffice member-screens 'ทางเข้าจาก LINE' badge computed lineEntry from line_entry and workspace — neither field has ever existed in CustomerAppResponse — so it read 'not connected' for every company since day one, while the suite stayed green"
metadata:
  node_type: memory
  type: reference
---

2026-09-22, `frontend/oc2plus-linecrm-frontend-backoffice`.

`src/services/customer-app/mapper.ts` did:

```ts
lineEntry: r.line_entry ?? (r.workspace?.id ? 'connected' : 'not_connected')
```

`CustomerAppResponse` in backoffice-api has **neither** `line_entry` nor
`workspace`, on `main` or `develop` — only `binding_status`, `company_id`,
`member_app_base_url`, `pages`, `slug`. So the expression fell through to
`'not_connected'` **always, for every company, forever**. The badge had never
once said "connected".

Meanwhile the response *did* carry `binding_status` with the right value — it
was simply never declared in `RestCustomerAppResponse`, so the mapper could not
see an answer that had already reached the browser. A comment in the file even
referred to `binding_status`.

Why 5 existing spec cases stayed green: **every one of them fed `line_entry` or
`workspace`** — fields the server never sends. The fixtures and the code shared
one author and one belief, which is exactly
`.claude/rules/verify-at-the-boundary.md` §1. Building one fixture from a
captured real response is what exposed it.

Fixed in MR !655 →
https://gitlab.sellsuki.com/sellsuki/oc2plus/line-crm/frontend/oc2plus-linecrm-frontend-backoffice/-/merge_requests/655

Separate root cause for the same screen looking wrong:
[[reference_bola_sweep_binds_without_an_owner]].
