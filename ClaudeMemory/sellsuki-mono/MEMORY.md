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
| [feedback_subagent_model_tiering.md](feedback_subagent_model_tiering.md) | feedback | Tier sub-agent models by difficulty (haiku scan / sonnet research / opus verify) — don't run everything on the big model |
| [project_pis_frontend_local_testing.md](project_pis_frontend_local_testing.md) | project | pis-frontend points at dev-th STAGING by default; how to wire UI → local pis-api for testing |
| [feedback_ff_only_force_push_ok.md](feedback_ff_only_force_push_ok.md) | feedback | PIS repos are FF-only → rebase+force-push FEATURE branches is OK (not main) |
| [reference_ccs_config_namespaces.md](reference_ccs_config_namespaces.md) | reference | CCS config namespaces: patona-seller-center-user/store, sellsuki-global-user — schemas + missing patona-provider-company |
| [reference_pat_jira_project.md](reference_pat_jira_project.md) | reference | Patona Jira project key PAT, board 71 — company management epics PAT-2448 to PAT-2452 |
| [project_provider_frontend_backend_routing.md](project_provider_frontend_backend_routing.md) | project | provider-management-frontend → CCS backend (8092) not management-backend; store CRUD/file upload/CCS config proxy only in management-backend |
| [project_bola_auth_mode_deployment.md](project_bola_auth_mode_deployment.md) | project | BOLA auth: SaaS=kratos, multi-tenant=local_jwt; AUTH_MODE/VITE_AUTH_MODE env; deploy env in SRE repo not monorepo |
| [project_overmind_restart_quirk.md](project_overmind_restart_quirk.md) | project | overmind: use `overmind restart <name>` for dead proc; `start <name> -D` from CLAUDE.md doesn't work |
| [project_oc_epic_backlog_triage.md](project_oc_epic_backlog_triage.md) | project | OC board ~127 non-Done epics (stale clutter); 26 dead-shells closed 2026-06-22; MCP Jira cross-project contamination bug |
| [project_oc_bola_domain_boundary.md](project_oc_bola_domain_boundary.md) | project | OC2Plus×BOLA boundary (OC-4200): LIFF register = 2-layer split; membership=OC2Plus, LINE/LIFF=BOLA; card re-map |
| [reference_oc2plus_member_frontend.md](reference_oc2plus_member_frontend.md) | reference | OC2Plus customer member-register LIFF frontend = existing repo oc2plus-linecrm-frontend-member (GitLab 2yr); OC-4245 goes there; ls-remote GitLab not just local submodules |
| [project_oc4267_standalone_no_qms.md](project_oc4267_standalone_no_qms.md) | project | OC-4267 descoped to standalone binding (no QMS/quota/deactivate); enum pending/active/failed; QMS reconnects as follow-up; code 4c8a2e4 must strip QMS |
| [feedback_oc_pat_board_ownership_rule.md](feedback_oc_pat_board_ownership_rule.md) | feedback | OC↔PAT board rule: ขารับ (OC2Plus serves/consumes) = OC board; ขา request/produce (Patona calls/publishes) = PAT board; duplicates → canonical per rule, other closed+linked |
| [project_sukipay_audit_log.md](project_sukipay_audit_log.md) | project | SukiPay audit: table transaction_audit_logs (not trails); PAT-2286 rewrite card set A-F; PAT has no Tech Story type |
| [project_bola_kratos_sso_staging.md](project_bola_kratos_sso_staging.md) | project | BOLA staging Kratos SSO: API must be on *.staging-th.sellsuki.com (cookie domain); Emissary cfg in sre/configuration/api-gateway; staging deploys only from main+manual |
| [project_bola_migrations_jsonb.md](project_bola_migrations_jsonb.md) | project | bola runs migrations on boot (Fatal→crashloops all pods, blocks deploys); custom_fields is TEXT not jsonb (cast ::jsonb before ->>); Postgres-only migrations skip SQLite tests |
| [project_control_tower.md](project_control_tower.md) | project | Control Tower site at docs/control-tower/ — ecosystem/roadmap/connection-map dashboard; progress derived from epic-index.md; run via launch.json "control-tower" :4480 |
| [reference_env_urls.md](reference_env_urls.md) | reference | Env URL convention: dev=.dev-th, staging=.staging-th, prod=none; gateway api.{suffix}.sellsuki.com/{svc}/v1; AMS/admin/PIS/RAG/GitLab anchors |
| [project_pat_epic_links_unwired.md](project_pat_epic_links_unwired.md) | project | PAT epic↔child `parent` links NOT wired (grouped by [tag]+prose only); roadmap/epic-index drift; 174 orphans; 6 missing epics; full doc in .claude/knowledge/pat-epic-gap-analysis.md |
| [project_sukipay_void_rename.md](project_sukipay_void_rename.md) | project | SukiPay transaction state rename VOID_PREPARED→COMPENSATING, VOID→COMPENSATED — EXECUTED in Jira(12)+Outline(11); code+migration pending dev |
| [reference_jira_editissue_adf_breakage.md](reference_jira_editissue_adf_breakage.md) | reference | editJiraIssue markdown breaks embedded images/Figma smartlinks + escapes checkboxes; tables/headings/code survive — verify format after rewrite |
| [project_sukipay_refund_cluster.md](project_sukipay_refund_cluster.md) | project | PAT refund cluster roles (2424 produce/2289 execute/2036 read-only) + recurring attribution bug + OMS↔SukiPay sync/async + 3 state layers |
| [project_quota_not_feature_gate.md](project_quota_not_feature_gate.md) | project | quota-management = usage metering, NO hasFeature() boolean; feature-gate ต้องใช้ plan/capability/admin-override ไม่ใช่ quota |
| [project_oms2_plan_gaps_2026q3.md](project_oms2_plan_gaps_2026q3.md) | project | Gap analysis 2026-07-03: OMS 2.0 ไม่มี SP สักใบ (waves ≠ schedule), 2244↔2057 lock ไม่มีจริง, 2057 เป็นเปลือก, RAG P0 ไม่มี epic, parcel 19 ใบ superseded ค้างใน 2241 |
| [reference_pat_board_sprints.md](reference_pat_board_sprints.md) | reference | PAT sprint truth = board 71 JQL (customfield_10020) — ห้ามใช้ epic-index เป็น sprint state; PAT-2038/2036 identity เคยผิดใน index |
| [project_user_pain_evidence_gap.md](project_user_pain_evidence_gap.md) | project | องค์กรไม่มี user pain research จริง — QA bugs คือ evidence ที่แข็งสุด; address-form cluster + OC loyalty redemption = pain ไร้ epic |
| [project_loyalty_point_cluster.md](project_loyalty_point_cluster.md) | project | Loyalty cluster OC-4295/4297/4335/4339 — purchase event+conversion, welcome, QR-receipt pull model (rate ณ purchased_at), clawback; QR = URL render client-side |
| [feedback_card_user_story_flows.md](feedback_card_user_story_flows.md) | feedback | การ์ดห้ามเป็น technical concept ล้วน — ต้องมี User Story + Flow create/edit/runtime, technical scope เป็น supporting section |
| [feedback_no_scope_change_in_sprint.md](feedback_no_scope_change_in_sprint.md) | feedback | ห้ามแก้ scope การ์ด In Progress ใน sprint — enhance เปิดการ์ดใหม่ blocked-by แทน; ถ้าแตะแล้วให้ revert |
| [reference_messaging_backend.md](reference_messaging_backend.md) | reference | Messaging กลาง = sellsuki-messaging-backend (OTP/SMS Thaibulk, extensible provider) — email OC-4343 ทำที่นี่; sellsuki-service-messaging ตายแล้ว |
| [project_oms2_decouple_decision.md](project_oms2_decouple_decision.md) | project | OMS 2.0 เลือก Decouple (PAT-2540); children 2546-2550 (derivation/generator/templates/guard/FE) + slices 2292/2293 เป็น consumer (de-dup แล้ว) |
| [project_customer_app_program.md](project_customer_app_program.md) | project | Customer App program แทน posh-medica: 5 epics label customer-app-program (OC-4344/4349/4355/4359 + OC-2743); BFF ที่ member-api; web login ใน MVP; DoR blockers 4 ใบ |
| [project_invite_multiapp_chain.md](project_invite_multiapp_chain.md) | project | Invite→app chain: PAT-2553 return_to email, PAT-2554 deep-link, OC-4371 OC2Plus bridge; BOLA=kratos cookie |
