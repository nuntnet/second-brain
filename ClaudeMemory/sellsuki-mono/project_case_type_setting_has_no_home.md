---
name: project_case_type_setting_has_no_home
description: RULED 2026-09-02 — one case_type per workspace, set at creation, immutable after; catalog lives in chat-core not CCS (undocumented deviation), so AI-152's AC is impossible
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


## เคาะแล้ว 2026-09-02 — 1 workspace = 1 case_type, ตั้งตอนสร้าง แก้ไม่ได้เลย

The plan contradicted itself: §5.13.1 rule 4 constrains a **Case** ("1 case = 1
case_type immutable") and `AllowedTypes` is a list of up to 50, while §5.11 says
*"Workspace = app instance: company เดียวมี workspace ขาย + workspace HR ได้"*.
**§5.11 wins.** A second template is a second workspace. There is **no editor** —
the value is chosen at workspace creation and never again.

The runtime already only supported that shape, which is what decided it:
- `resolveInboundCase` opens every Case **without passing a case_type** → every
  Case a customer starts gets `DefaultType`; **no classifier exists anywhere**
- `checkpoint_playbooks` is keyed by `workspace_id` + unique `is_current` ⇒
  **one playbook per workspace**
- `checkpoint.Compute(playbook, facts, language)` **never receives the Case's
  case_type** ⇒ a second allowed type is served by the first type's playbook,
  silently (the mismatch AI-128 exists to catch)

Landed: plan §5.13.1 now carries both the deviation note and this ruling
(monorepo commit `226ed0d`); the misleading "future E8 admin editor" comment is
gone (chat-core MR !52, merged `5c9a2866`).

**Still open, needs a decision:** whether to narrow `AllowedTypes []string` to a
single value (it belongs to AI-126, which is Done), and which feature keeps the
word **"goal"** — AI-115's conversion target, the checkpoint position (i18n says
*"the conversation-goal position"*), or AI-152's template component.

**Still to build:** a template picker on the workspace creation form (it has only
name + FB page id + token today) and a write path — `CaseRepository` still has no
write method at all.
