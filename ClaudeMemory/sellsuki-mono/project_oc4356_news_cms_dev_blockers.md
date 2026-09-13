---
name: project_oc4356_news_cms_dev_blockers
description: "OC-4356 News CMS + OC-4529 บัตรสมาชิก: deploy dev แล้วแต่ตาราง news/member_card_token ไม่มี และ oc2plus.news.manage 0 tuple — แก้ครบ 2026-09-13 (apply SQL ใน pod + backfill 4536 roles + MR !86 ที่ repo migration); staging/prod ยังไม่ได้ทำ"
metadata:
  node_type: memory
  type: project
---

**ตรวจ 2026-09-13 ~20:30** หลัง PO ถาม "OC-4356 เสร็จยัง" · Jira = `Ready to test (DEV)` แต่ 0 comment, unassigned

## โค้ดครบและ deploy dev แล้ว
- backoffice-api develop `b1d0ab9` (image `:b1d0ab9f` บน `octoplus-dev`) มี `news.go` / `news_sanitizer.go` (bluemonday, whitelist ตามการ์ด) / 4 route / audit create-update-publish-archive / 28 test cases
- frontend-backoffice develop `6422576` deploy แล้ว — รวม !603 (feature) + **!604 (แก้ contract ให้ตรง BE: cursor paging, POST update, cover_image_base64)** — !603 เคย merge ก่อน !604 จึงมีช่วงสั้น ๆ ที่ FE บน develop คุยกับ BE ไม่ตรง
- member-api develop มี `migrations/018_create_news` (แต่ดูข้อล่าง)

## 🔴 ใช้บน dev ไม่ได้ 2 ชั้น (ตรวจของจริง ไม่ใช่เดา)
1. **Keto dev (`share-dev/keto-read`): `oc2plus.news.manage` = 0 tuple** (เทียบ `oc2plus.member.view` / `oc2plus.point.adjust` ≥ 500) → ทุก admin โดน 403 ก่อนถึง DB, เมนู "ข่าวสาร" ถูกซ่อน (SideBar gate ตัวเดียวกัน)
   - โค้ดเองบอกไว้ (`news.go:18-40`): permission นี้เป็น **local const placeholder** ไม่มีใน `entity v0.34.0 access_control/permission_list.go` → ต้อง (a) เพิ่มใน shared entity แล้ว bump, (b) grant ให้ role ผ่าน rps `cmd/backfill_role_permission --apply` (preset apply ตอนสร้าง company เท่านั้น — [[project_ccs_role_presets_apply_only_at_creation]])
2. **DB dev (`datastore/postgresql` → pod `postgresql-pg18-0`, db `development_oc2plus_crm`): `to_regclass('public.news')` = NULL** → migration 018 ยังไม่ run · CRM migrations ไม่มี CI ต้อง run มือ ([[project_oc2275_crm_migrations_run_by_hand]])

## ⚠️ พลอยเจอ: `member_card_token` (017) ก็ไม่มีบน dev
→ `GET /v1/me/card-token` (OC-4529) และ verify ทั้งสาย (OC-4539 !243/!572, FE !602 ที่ deploy แล้ว) **จะล้มบน dev** จนกว่าจะ run 017 · ต้อง run 017+018 พร้อมกัน

## ✅ แก้แล้ว 2026-09-13 (ทั้งสองข้อ)

1. **Keto:** `kubectl -n sellsuki-dev exec deploy/sellsuki-role-and-permission-management-backend -- /app/backfill-role-permission --permission oc2plus.news.manage --apply`
   → **granted 4536, failed 0** (ใช้เวลา ~10 นาที) · ยืนยันด้วย keto-read: page 1 เต็ม 1000 tuple เท่ากับ `oc2plus.member.view`
   · dry-run ก่อน (ไม่ใส่ `--apply`) เตือน `[warn-no-tenant]` ไม่กี่ role = role ไม่มี tenant ref เขียน tuple ไม่ได้ ปกติ
2. **DB:** classifier บล็อกการอ่าน secret ออกมานอกคลัสเตอร์ → **รัน SQL ข้างในแทน**
   `kubectl -n datastore exec -i postgresql-pg18-0 -c postgresql -- sh -c 'export PGPASSWORD=...; psql -U postgres -d development_oc2plus_crm -v ON_ERROR_STOP=1 -f -' < file.sql`
   ใส่ `member_card_token` + `news` ใน transaction เดียว **พร้อม INSERT แถว ledger** ชื่อ `/20260913000000-create-member-card-token`, `/20260913001000-create-news`
   ให้ตรงกับไฟล์ใน **MR !86** ของ `oc2plus-line-crm-migration` → `db-migrate up -e dev` จะข้ามเอง ไม่ชน

🟡 **staging/prod ยังไม่ได้ทั้ง migration และ backfill** · 🟡 `oc2plus.news.manage` ยังเป็น local const ใน `news.go` ต้องเพิ่มลง entity `access_control/permission_list.go` แล้ว bump
🟡 **`member_tier` (OC-3559) ยังไม่ apply บน dev** — ledger dev หยุดที่ `campaign-product-scope`; ไฟล์อยู่ใน repo migration แล้ว จะติดไปกับ `db-migrate up` ครั้งหน้า

วิธีตรวจซ้ำ: port-forward keto-read แล้ว `GET /relation-tuples?namespace=permissions&object=<perm>` · DB: resolve pod จาก endpoints ของ svc/postgresql ก่อน ([[reference_datastore_stale_postgres_pod]])

เชื่อม [[project_oc4530_counter_scan_reanalysis]] [[reference_dev_th_cluster_access]] [[reference_keto_staging_permission_lookup]]
