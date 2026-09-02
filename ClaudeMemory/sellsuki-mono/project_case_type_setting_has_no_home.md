---
name: project_case_type_setting_has_no_home
description: case_type config is stored in chat-core with no write path and no editor; AI-152 assumes it lives in CCS config and forbids migrations, which git already contradicts
metadata:
  type: project
---

There is **no way to change a workspace's `case_type` catalog** in the running
product, and the card that depends on doing so assumes a storage location that
does not exist.

**What the code actually does** (`backend/sellsuki-chat-core`, `origin/main`):
- `src/entity/case_/case_type_config.go` — `CaseTypeConfig{WorkspaceID,
  AllowedTypes, DefaultType, Version}`, `MaxAllowedCaseTypes = 50`,
  `DefaultCaseType = "sales_lead"`
- table `case_type_configs` in chat-core's OWN Postgres (workspace_id PK,
  allowed_types comma-separated)
- seeded by migration `0064_add_case_engine`, corrected by
  `0066_case_type_sales_lead` (`insurance_lead` → `sales_lead`)
- `CaseRepository` exposes **`GetCaseTypeConfig` only — no write method**
- no use case, no route, no OpenAPI path, no frontend

AI-126's own comment says why: *"This card ships no HTTP surface to edit it
(out of this card's scope — a future E8 admin editor…)"*. **No such E8 card
exists** — an E8 sweep found 14 cards and none is this.

**The contradiction to raise before anyone builds it.** AI-152 (`[E12][Template]
HR/Internal Solution Template v1 (case_type = hr_request)`, To Do) states:
- "ที่เก็บ config = **AI-19** (CCS config namespace)"
- AC: add `hr_request` **by config only** — *"ไม่มี commit แก้ชั้น core และไม่มี
  migration — พิสูจน์จาก git"*

But `grep case_type` over `backend/sellsuki-central-control-backend/src`
returns **nothing**, chat-core never reads case types from CCS, and changing a
case_type value has **already required two migrations** (0064, 0066) — exactly
what the AC forbids and asks git to prove. As written, that acid test cannot
pass for storage reasons, not engineering ones.

Three candidate owners, and picking one is a product decision:
1. an **E8 admin editor** (what the code comment says) — no card
2. **AI-19 / CCS config namespace** (what AI-152 says) — needs the store moved
   first, and AI-19 has an open decision outstanding since 2026-08-01
3. **AI-146 Solution Template catalog** — the code itself says *"case_type is a
   SOLUTION TEMPLATE"*, which is AI-146's subject

**Why:** `case_type` is the Solution Template key, so this blocks AI-152, which
in turn blocks AI-146 (its dependency table names AI-152 as the unblocking
card). A whole E12 chain rests on a setting nothing can write.

**How to apply:** never say "case type is configurable per workspace" without
adding that only a migration or direct SQL can change it today. Relates to
[[project_ai146_onboarding_wizard_not_started]],
[[reference_ccs_config_namespaces]] and [[reference_ai_board_stale_cards]].
