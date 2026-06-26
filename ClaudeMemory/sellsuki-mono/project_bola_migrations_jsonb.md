---
name: project_bola_migrations_jsonb
description: bola-backend runs DB migrations on boot (Fatal on fail → crashloops all pods); custom_fields is TEXT not jsonb — must cast ::jsonb before ->>
metadata: 
  node_type: memory
  type: project
  originSessionId: 8ea4e16e-492a-4d1e-8bcb-146ecab6fad2
---

Two bola-backend (back-office-of-line-api-backend) gotchas that caused a full staging deploy outage (BOLA-264 migration 0121):

**1. Migrations run on boot, Fatal on failure.** `cmd/bola_server/helper.go:setupGorm` calls `runMigrationsOnStartup(db)` and does `slog.Fatal("Database migration failed")` if it errors. This runs for the HTTP server AND every cron (segment-refresh, lon-job, rich-menu-sync, broadcast). So a bad migration crashloops EVERY pod → readiness never green → `helm upgrade --atomic --wait` times out → deploy auto-rolls-back to the previous image. The deploy pipeline (golang-th) does NOT run migrations separately — boot is the migration mechanism. A broken migration on main blocks ALL bola-backend deploys until fixed (fix-forward).

**2. `custom_fields` is a TEXT column, not jsonb.** Defined `custom_fields TEXT NOT NULL DEFAULT '{}'` (followers, contact_profiles, etc.). To use JSON operators you MUST cast: `custom_fields::jsonb ->> 'key'`. Convention is everywhere — customer_repository, segment_repository, migration 0099 (contact_profiles GIN index comments "custom_fields is stored as text in Postgres, so we cast to jsonb"). Migration 0121 forgot the cast → `operator does not exist: text ->> unknown (SQLSTATE 42883)` → boot Fatal. Fixed in MR !101.

**Why tests passed but prod failed:** unit/integration/e2e run on SQLite, and migration 0121 explicitly skips on SQLite (`tx.Dialector.Name() == "sqlite"`). PostgreSQL-only migrations are invisible to the test suite — verify them against Postgres before merge.

Backend staging-th deploys automatically on `main` (`deploy_staging_th_arm`, not manual — unlike the frontend which is manual). Cluster: `eks_staging_th-cluster` via Teleport (`tsh login` + `tsh kube login eks_staging_th-cluster`; cert expires ~hourly). Related: [[project_bola_kratos_sso_staging]]
