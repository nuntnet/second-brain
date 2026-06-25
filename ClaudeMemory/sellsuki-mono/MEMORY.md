# Memory Index

| File | Type | Description |
|------|------|-------------|
| [project_merchant_portal.md](project_merchant_portal.md) | project | The merchant portal is shipmunk-frontend (not sellercenter-frontend) |
| [reference_nas_ds1.md](reference_nas_ds1.md) | reference | SSH to Synology NAS DS1 via Tailscale only: ssh -p 2022 nunt@100.65.67.121 |
| [project_helio.md](project_helio.md) | project | Helio control-tower platform — arch, component status, Phase 0 hermes↔n8n bridge blocker |
| [reference_secondbrain_vault.md](reference_secondbrain_vault.md) | reference | Obsidian vault ~/SecondBrain — Claude memory symlinked in; auto-syncs via launchd every 15 min |
| [user_language_thai.md](user_language_thai.md) | user | User writes in Thai and prefers Thai responses |
| [reference_bola_jira_project.md](reference_bola_jira_project.md) | reference | BOLA tickets are in Jira project BOLA (id 10126), not LR as repo config claims |
| [project_bundle_in_catalog.md](project_bundle_in_catalog.md) | project | Bundle/set product lives in catalog-service (multi-catalog), not pis-api |
| [project_segment_export_static_snapshot.md](project_segment_export_static_snapshot.md) | project | BOLA static segment export must read from snapshot table, not re-eval rule live |
| [reference_oc2plus_jira_project.md](reference_oc2plus_jira_project.md) | reference | OC2Plus Jira project key is "OC" (id 10001) — repo config says TBD |
| [project_oc4207_line_optional_design.md](project_oc4207_line_optional_design.md) | project | OC-4207: empty string = no LINE (not *string); affects OC-4241, OC-3339, OC-3340, OC-3341 |
| [project_qms_ui.md](project_qms_ui.md) | project | QMS UI (OC-4257) → provider-management-frontend + management-backend proxy; no ListAssignPlans gRPC |
| [project_ms687_reserve_needs_company_location.md](project_ms687_reserve_needs_company_location.md) | project | MS-687 OMS↔inventory reserve BLOCKED — needs company+location; orders don't carry it yet |
| [feedback_decisive_deep_execution.md](feedback_decisive_deep_execution.md) | feedback | Be sharper/decisive — investigate & act, don't offer ก/ข/ค menus |
| [project_pis_frontend_local_testing.md](project_pis_frontend_local_testing.md) | project | pis-frontend points at dev-th STAGING by default; how to wire UI → local pis-api for testing |
| [feedback_ff_only_force_push_ok.md](feedback_ff_only_force_push_ok.md) | feedback | PIS repos are FF-only → rebase+force-push FEATURE branches is OK (not main) |
| [reference_ccs_config_namespaces.md](reference_ccs_config_namespaces.md) | reference | CCS config namespaces: patona-seller-center-user/store, sellsuki-global-user — schemas + missing patona-provider-company |
| [reference_pat_jira_project.md](reference_pat_jira_project.md) | reference | Patona Jira project key PAT, board 71 — company management epics PAT-2448 to PAT-2452 |
| [project_provider_frontend_backend_routing.md](project_provider_frontend_backend_routing.md) | project | provider-management-frontend → CCS backend (8092) not management-backend; store CRUD/file upload/CCS config proxy only in management-backend |
| [project_bola_auth_mode_deployment.md](project_bola_auth_mode_deployment.md) | project | BOLA auth: SaaS=kratos, multi-tenant=local_jwt; AUTH_MODE/VITE_AUTH_MODE env; deploy env in SRE repo not monorepo |
| [project_overmind_restart_quirk.md](project_overmind_restart_quirk.md) | project | overmind: use `overmind restart <name>` for dead proc; `start <name> -D` from CLAUDE.md doesn't work |
| [project_oc_epic_backlog_triage.md](project_oc_epic_backlog_triage.md) | project | OC board ~127 non-Done epics (stale clutter); 26 dead-shells closed 2026-06-22; MCP Jira cross-project contamination bug |
| [project_oc_bola_domain_boundary.md](project_oc_bola_domain_boundary.md) | project | OC2Plus×BOLA boundary (OC-4200): LIFF register = 2-layer split; membership=OC2Plus, LINE/LIFF=BOLA; card re-map |
| [project_sukipay_audit_log.md](project_sukipay_audit_log.md) | project | SukiPay audit: table transaction_audit_logs (not trails); PAT-2286 rewrite card set A-F; PAT has no Tech Story type |
