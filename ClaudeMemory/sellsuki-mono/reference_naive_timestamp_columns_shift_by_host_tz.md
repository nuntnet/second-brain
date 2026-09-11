---
name: reference-naive-timestamp-columns-shift-by-host-tz
description: "Go time.Now() into a `timestamp without time zone` column stores the host's wall clock — dates render +7 (a day late) on a dev Mac and are correct in the cluster"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f1e1e4b4-53e4-4459-99e1-ea7fc29faaa9
  modified: 2026-09-11T14:02:37.488Z
---

OC2Plus tables use `timestamp without time zone`. A Go `time.Time` written
there keeps the **wall clock** and drops the offset. So on a host in +07:

```
register at 17:41 ICT  →  stored "17:41"  →  read back as 17:41 UTC
                       →  browser renders 00:41 the NEXT DAY
```

The backoffice member card showed "เป็นสมาชิกเมื่อ 12 ก.ย." for someone who
registered on the 11th. It reads as a frontend date-formatting bug and is not
one — Luxon's `DateTime.fromISO` is doing exactly the right thing with the
wrong input.

**Why nobody caught it:** staging, production and CI all run UTC containers,
where `time.Now()` is already the right answer. It is wrong only on developer
machines — which is precisely where screens get eyeballed.

**How to confirm in one query** — if `created_at` is ahead of `now()`, or equal
to your local wall clock rather than `date -u`, it was written naive:

```sql
SELECT now() AS db_now_utc, created_at FROM member ORDER BY created_at DESC LIMIT 3;
```

Fixed in member-api 2026-09-11 (`cmd/generics_server/main.go`) with
`func init() { time.Local = time.UTC }` — pinned once rather than `.UTC()` at
~20 call sites, because those include **comparisons** (`expire_at >= now`) as
well as writes and a half-converted set is worse than none: a UTC write
compared against a local now expires sessions hours early.

⚠️ **Other Go services in this monorepo were not audited for this.** Same
pattern, same column type — check before trusting a timestamp rendered on a
local stack.

Rows written before the fix stay shifted; correct them with
`created_at - interval '7 hours'` only where you know a Go path wrote them
(SQL-seeded rows used `CURRENT_TIMESTAMP` and are already UTC).

See also [[reference_oc2plus_register_did_not_mint_a_session]].
