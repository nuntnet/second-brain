---
name: project_sukipay_audit_log
description: "SukiPay audit log — canonical table/gRPC names, PAT-2286 rewrite card set, PAT has no Tech Story type"
metadata: 
  node_type: memory
  type: project
  originSessionId: 70f89b38-8a91-43dd-99c2-8412adb43abe
---

SukiPay audit infra (sellsuki-pay-backend): canonical names are `TransactionAuditLogService.ListTransactionAuditLogs` + table **`transaction_audit_logs`** (migration 0013 renamed it from `transaction_audit_trails` → logs). Knowledge docs / PAT-2049 still say `...trails`/`...AuditLogs` inconsistently — `...logs` is ground truth. Table has NO tenant column; company-scoping must JOIN `payment_transactions.tenant_id` (added migration 0015). `transaction_id` is part of composite PK on a pg_partman monthly-partitioned table — verify PK before any nullable migration.

**PAT-2286 rewritten (2026-06-25)** from "audit section in Transaction Detail (single txn)" → standalone company-scoped audit log page in CCS3 (sellsuki-company-management-frontend). Card set under epic PAT-2060, relates PAT-2049:
- PAT-2485 (A) BE cross-txn query + company scoping + migration — **do first, unblocks all**
- PAT-2483 (C) management-backend proxy (X-Company-Id→ref_id, Keto, anti-spoof)
- PAT-2484 (B) extend audit capture (channel config/provider bank/refund) via generic entity_type/entity_id columns
- PAT-2286 (D) FE page
- PAT-2482 (F) export CSV/Excel — FUTURE/P2, not in sprint

**Project PAT has no "Tech Story" issuetype (id 10371)** — backend cards fall back to Task/Story. See [[reference_pat_jira_project]].
