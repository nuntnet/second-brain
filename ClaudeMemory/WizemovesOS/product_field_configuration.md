---
name: Field Configuration Strategy (Admin-Only)
description: WizemovesOS uses admin-only field configuration — no user-facing UI for custom fields
type: project
---

## Decision
**Custom fields are admin-only** — configured via database migrations/seeds, NOT through Settings UI.

## Rationale
- Simpler implementation (no CRUD UI for fields needed)
- Safer — avoids accidental field deletion losing data
- Legacy Project Center didn't have user custom fields either
- Aligns with typical SaaS multi-tenant pattern: operators configure, users use

## What This Means
- **Users CANNOT:**
  - Add custom fields in Settings UI
  - Delete or modify field configuration
  - Create per-account field variants

- **Admins CAN:**
  - Add fields via `supabase/seed.sql` INSERT into app_settings
  - Modify fields via database migrations
  - Deploy field changes across all customer instances

## Implementation
- Settings UI shows **read-only** field configuration (if useful for debugging)
- Field schema changes happen at operator level
- All Sellsuki accounts use the same global field set (unless custom seeding per client)
