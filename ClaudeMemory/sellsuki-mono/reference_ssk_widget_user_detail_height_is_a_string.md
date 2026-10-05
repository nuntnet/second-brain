---
name: reference_ssk_widget_user_detail_height_is_a_string
description: ssk-widget-user-detail picks its row count from widgetHeight with a STRING switch ("4"→4, "6"→7, else 2) — binding a number shows 2 rows; ceiling is 7 rows
metadata:
  type: reference
---

Found 2026-10-05 (OC-3126). `@sellsuki-org/sellsuki-components` `ssk-widget-user-detail` renders
`rowItems.slice(0, maxItemsToShow)` where `maxItemsToShow` is `switch(this.widgetHeight){case "4":4; case "6":7; default:2}`.

- The OC2Plus backoffice member page bound `:widgetHeight="… ? 7 : 6"` (a number) → every member card showed only
  gender + birthdate; phone/email/occupation/address were silently cut for everyone. Fixed on OC-3126 branch (`widgetHeight="6"`).
- Max is **7 rows**. A row that makes it 8 is dropped from the end without a sign.
- Checking it: `shadowRoot.textContent` of the widget only shows the rows it drew; `rowItems` (property) holds them all.
  Nested ssk components have their own shadow roots — read the property, not the text, before concluding a row is missing.
