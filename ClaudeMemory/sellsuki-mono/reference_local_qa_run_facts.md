---
name: reference_local_qa_run_facts
description: รัน QA suite (Robot) บน local stack — ชื่อ DB จริง, venv, env var, เรทลิมิตที่บล็อกถาวร + วิธีล้าง, เซสชันเว็บแทรกลง DB
metadata:
  type: reference
---

(ตรวจ 2026-10-03) ใช้ `ENV=local` → `tests/api/variables/local/variables.py` (branch `refactor/local-run-variables` ของ `testing/oc2plus-line-crm-automate-testing`, ยังไม่ push) · worktree `scratchpad/qa-run`, venv `scratchpad/qa-venv` (ติดตั้งโดยข้าม `psycopg2` ตัวที่ต้อง build — มี `psycopg2-binary` แล้ว) · สคริปต์รัน `run1.sh <suite>`

- **DB จริงบน local:** central = `local_sellsuki_central` (ไม่ใช่ `sellsuki_central`), consent service อ่าน Mongo `develop-sellsuki-service-consent` (ไม่ใช่ `consent`) — เดาชื่อผิดทำให้ suite ล้มเป็นร้อย; ดูจาก `pg_database` / ดูว่า service ตอบอะไรจริง
- **Postgres cred:** อ่านจาก `docker-compose.yml` (POSTGRES_USER/PASSWORD) ใส่ env `POSTGRESQL_USER/PASSWORD` โดยไม่พิมพ์ · `libs/postgresql` มี override ตามนี้
- **เซสชันเว็บ (ไม่ต้องใช้ TEST_KEY):** แทรกแถว `session` ที่ `integration_id` NULL + token รู้ค่า → ยิง member-api ด้วย `Cookie: oc2plus_crm_session=<token>`
- **member-api `/me/point|campaigns` อ่าน DB ตรง ไม่ผ่านเกท consent** — ใช้ `/me/coupons` พิสูจน์เกท · ประตูทางเข้าคือ `ResolveCompanyBySlug` (ต้อง activated + opened to members)
- **🔴 เรทลิมิตโค้ด (3rdparty-api) บล็อกถาวร:** เกิน 10 req/s หรือ 60/min ต่อ caller → Redis `crm:ratelimit:block:<member>` TTL ~173 ปี + แถว `member_block` ("Block permanent…") → 429 "contact support" ตลอด · ล้าง: `docker exec sellsuki_mono-redis-1 redis-cli del crm:ratelimit:block:<member>` (ห้ามลบคีย์ `…:memberblocklistexists`) + ลบแถว `member_block` ของตัวเอง · เทสที่ยิงขนาน: ใช้ ≤3 racers ห่างกัน ~12s
- 3rdparty-api เชื่อ `X-User-Id/X-User-Kind` และ `X-Api-Key-Id/X-Company-Id/X-Api-Scope` ล้วน ๆ (scope แยกด้วยช่องว่าง) · route `/v2/openapi/*` ที่ไม่มี policy ตอบ 500 `unexpected_error`

ที่เกี่ยวข้อง: [[feedback_tests_derive_from_the_card_not_the_code]], [[feedback_comment_jira_card_when_loop_closes]]

## Added 2026-10-03 (OC-4340 / OC-2261 runs)
- **Consent acceptance WRITE cannot run on local**: consent service `/consentee/acceptance` needs a Mongo replica set (`CheckReady` reads `hello.setName`) + `CONSENT_ACCEPTANCE_WRITE_ENABLED`; local docker Mongo is standalone. 3rdparty `CONSENT_ACCEPTANCE_ENABLED=true` also moves the READ onto that endpoint → leave it off locally; seed the `consentee` row directly (`id`, `consentId`, `version`, `options[{id,status:'accepted'}]`, `referenceId:'member_<id>'`).
- The consent service caches consentees in Redis (`consentee-<consent>-<version>-member_<id>`): deleting the Mongo row leaves a stale "accepted" → purge those keys in fixtures.
- Consent doc ids must be > 0; acceptance idempotency key must be a canonical UUID.
- Public `GET /company/{slug}/consent-policy` needs `THIRDPARTY_INTERNAL_API_KEY` (member-api) = `INTERNAL_API_KEY` (3rdparty-api) or it is 502. OTP test mode needs `SECRET_TESTER_KEY` on 3rdparty-api + `X-Testing-Secret` header (`X-Testing-Reference-1` = ref_no, `X-Testing-Response-Mode` 2 = wrong pin). Start these via `overmind start -f Procfile.oc2plus3rd -D` with `OVERMIND_SOCKET=./.overmind-oc2plus3rd.sock` and the env prefixed (member-api: `-f Procfile.oc4344 -l member-api -s .overmind-oc2plus-member.sock -N`).
- member-api wire codes ≠ 3rdparty's: 403 `CONSENT_REQUIRED`, 404 `consent_not_configured`.
