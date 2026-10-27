---
name: oc2plus-saas-plan-epic-oc4763-decisions
description: "2026-10-16 PO decisions for Oc2plus SaaS plan/QMS epic OC-4763 (cards 4755-4765) - soft quotas, no active_until, billing_start outside QMS, quota set via API, catalog only in provider sellsuki"
metadata:
  node_type: memory
  type: project
  originSessionId: 846eb860-82e8-4469-8af8-09ce62f37397
  modified: 2026-10-27T09:43:18.502Z
---

Epic OC-4763 (cards OC-4759..4765, OC-4755, OC-4756, OC-4758) was rewritten 2026-10-16 around these PO decisions. Numbers (prices, quota amounts) are PROPOSALS awaiting BD, not agreed prices.

- Bill per company; bulk-buy discount is commercial, outside the system, same plan. Setup fee not in QMS. Annual = `annual_discount_percent` only (one plan). OTP cost is the customer's.
- Over-cap is allowed: all quotas start soft (`report`), sales bills more. Hard/soft lives on the quota itself (OC-4764 option ก; per-plan-item option ข rejected). Active members never block in v1.
- Assign plans WITHOUT `active_until` (QMS refuses usage writes after it; auto-renew cron has a do-not-enable TODO). QMS `PeriodStart` = revision created time, cannot be future-dated, so `billing_start` (= customer app go-live) is stored separately in docs/release/environments.yaml.
- Quotas are set by the platform team calling the management-backend/QMS API; CCS2 advanced quota page stays off (VITE_QMS_ADVANCED_QUOTA_AUTHORING=false). Confirmed by PO 2026-10-16 (it was my reading of a bare "ใช่").
- Catalog lives only in provider `sellsuki` for now (PO confirmed); Posh Medica (provider poshmedica) has no plan and must not be locked by OC-4747 enforcement.
- "เข้าแอป" = successful OTP login only; "unlimited" is interim 999999 because QMS truthiness treats -1 as insufficient.

**Why:** Jira cards and docs/oc2plus-saas-plan-catalog-scope.md hold the text, but the reasons behind each choice (QMS code facts) are easy to lose.
**How to apply:** before editing these cards or answering plan/QMS questions, read OC-4763 first; do not reintroduce active_until, hard-by-default, or per-provider catalogs without a new PO decision. Related: [[feedback_ask_decisions_with_full_context_not_bare_lists]].
