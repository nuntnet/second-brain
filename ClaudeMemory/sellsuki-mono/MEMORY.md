# Memory Index

## User & how-to-work feedback
- [User writes Thai](user_language_thai.md) — prefers Thai responses
- [Decisive execution](feedback_decisive_deep_execution.md) — investigate & act, don't just list options
- [Self-explaining UX](feedback_selfexplaining_ux.md) — ทุกหน้าต้องอธิบายตัวเอง
- [Subagent model tiering](feedback_subagent_model_tiering.md) — tier by difficulty haiku→opus
- [Card = user story](feedback_card_user_story_flows.md) · [No scope change in sprint](feedback_no_scope_change_in_sprint.md) · [QA guide required](feedback_qa_reproduce_guide.md) · [Ground claims file:line](feedback_ground_claims_file_line.md) — card-writing rules
- [Verify as user sees it](feedback_verify_as_the_user_sees_it.md) — real logged-in session, not curl
- [Search before GAP](feedback_search_before_declaring_gap.md) · [Verify absence](feedback_verify_absence_claims.md) · [head on grep = sampling](feedback_head_on_grep_is_sampling_not_verification.md) — never claim "no X" without full search
- [PMM voice](feedback_product_marketing_voice.md) — write KB as PMM, not auditor
- [Parallel git safety](feedback_parallel_sessions_git_safety.md) · [List MRs before opening](feedback_list_open_mrs_before_opening_one.md) — parallel-session discipline
- [Design handoff fidelity](feedback_design_handoff_full_fidelity.md) — implement ทั้งเฟรม; no-mock = data only
- [Fix must reach everyone](feedback_fix_must_reach_everyone.md) — enumerate consumers not covered
- [Central ≠ caller domain](feedback_central_service_no_caller_domain.md) — scope belongs to consumer
- [Entity pure / zod = DTO](feedback_entity_pure_zod_dto.md) — zod only at boundary
- [Report wrong cards, don't edit](feedback_report_wrong_cards_dont_edit.md) — surface contradictions; user decides
- [OC↔PAT board rule](feedback_oc_pat_board_ownership_rule.md) — ขารับ = OC board owns
- [OC2Plus merge target](feedback_oc2plus_merge_to_develop.md) — MRs → develop, never main
- [PIS FF-only push](feedback_ff_only_force_push_ok.md) — rebase+force-push is correct there

## Personal / machine / infra
- [Merchant portal](project_merchant_portal.md) — = shipmunk-frontend
- [Ch.Erawan Next](project_ch_erawan_next.md) — separate car-dealer site
- [CATS ATS](reference_cats_ats_system.md) · [NAS DS1](reference_nas_ds1.md) — in-house ATS on ds1; SSH via Tailscale `-p 2022`
- [Helio platform](project_helio.md) — control-tower arch
- [SecondBrain vault](reference_secondbrain_vault.md) — Obsidian ~/SecondBrain
- [Control Tower](project_control_tower.md) — docs/control-tower/
- [Env URLs](reference_env_urls.md) — dev=.dev-th, staging=.staging-th, prod=none
- [rtk rewrites commands](reference_rtk_git_output_filtering.md) — false output; verify via full binary path
- [Caddy host-networking](reference_caddy_host_networking_gotcha.md) — refused → force-recreate
- [Browser surfaces](reference_browser_surfaces_this_workspace.md) — pane reaches localhost; never npm install member repo
- [Outline VPN blocker](reference_outline_mcp_vpn_blocker.md) — use publish-to-outline.py
- [Harness classifier blocks](reference_harness_classifier_blocks_secrets_and_mutations.md) — chat "yes" doesn't unlock
- [Overmind restart quirk](project_overmind_restart_quirk.md) · [Local bola overmind](reference_local_bola_own_overmind_socket.md) — restart for dead services; .overmind-bola.sock
- [dev-th access](reference_dev_th_cluster_access.md) · [Teleport kills dev-th](reference_teleport_session_kills_devth_access.md) — kubectl+Keto lookup; EOF = session หมด
- [Monorepo remotes](reference_monorepo_no_origin.md) — glab-base→BOLA is a trap
- [Mainline ≠ main](project_monorepo_mainline_is_not_main.md) — chore/ai-mvp-local-run is mainline
- [codegraph projectPath](reference_codegraph_context_needs_projectpath.md) — else wrong service
- [Background agent resume](reference_background_agent_resume_patterns.md) — 529 kills resume; nudge "จบในเทิร์นเดียว"

## Jira
- [BOLA Jira](reference_bola_jira_project.md) · [OC2Plus Jira](reference_oc2plus_jira_project.md) · [Patona Jira](reference_pat_jira_project.md) — BOLA id 10126 · OC id 10001 · PAT board 71
- [Jira MCP quirks](reference_jira_mcp_search_quirks.md) · [editIssue ADF break](reference_jira_editissue_adf_breakage.md) · [Sprint ids global](reference_jira_sprint_ids_not_contiguous.md) — no parallel calls; markdown breaks ADF; wrong sprint id files silently
- [PAT sprint truth](reference_pat_board_sprints.md) · [PAT epic links unwired](project_pat_epic_links_unwired.md) — customfield_10020; grouped by label

## Git / CI gotchas
- [FF merge reverts](reference_fast_forward_merge_silently_reverts.md) · [Silent semantic break](reference_silent_semantic_merge_break.md) · [Parallel dup symbols](reference_parallel_sessions_duplicate_symbols.md) — clean merges lie; dry-run
- [Submodules shallow](reference_submodules_are_shallow_clones.md) · [git @{u} false zero](reference_git_upstream_false_zero.md) — unshallow first; compare origin/<b>..HEAD
- [.bak restore drops comments](reference_bak_restore_drops_comments.md) — check diff for deletions
- [GitLab Go module CI](reference_gitlab_private_go_module_ci.md) · [Review-bot targets](reference_gitlab_review_bot_targets.md) · [Dead 'staging' runner](reference_dead_staging_runner_tag.md) · [glab ci stale](reference_glab_ci_status_stale_pipeline.md) — CI traps

## Test gotchas
- [testify default wins](reference_testify_permissive_default_wins.md) · [Timing concurrency tests](reference_timing_dependent_concurrency_tests.md) · [Turbo false green](reference_turbo_cache_crosssession_false_green.md) — tests that pass on broken code
- [Lit/React SSR hollow](reference_lit_react_node_condition_hollows_tests.md) · [DS testId property](reference_ds_testid_is_a_property.md) · [Node 25 jsdom](reference_node25_localstorage_jsdom_conflict.md) — DS/jsdom traps; machine has only Node v25

## Go / DB gotchas
- [GORM AutoMigrate 2nd boot](reference_gorm_pgx_libpq_automigrate.md) · [GORM Updates drops false](reference_gorm_updates_drops_false.md) — needs 3× boot + DB round-trip tests
- [Lease/claim bug class](reference_lease_claim_ownership_bug_class.md) · [PG partial-index ON CONFLICT](reference_pg_partial_index_onconflict_generic_plan.md) — double-processing; 42P10
- [Ambiguous 404](reference_ambiguous_404_fail_open.md) · [Auth behind own guard](reference_auth_endpoint_behind_own_guard.md) — branch on error_code; 40ms tell
- [Kafka silent failure](reference_kafka_silent_publish_failure.md) — missing KAFKA_SERVERS

## Identity / CCS / rps
- [Identity infra SPOF](reference_shared_identity_infra_singleton.md) — kratos/keto/hydra 1-replica
- [Local identity loop](reference_local_identity_hardcode_loop.md) · [Local Kratos debugging](reference_local_kratos_identity_debugging.md) — forward_auth fix; read backend logs
- [Local i18n/config seed](reference_local_i18n_config_seeding.md) — pulls staging public API
- [CCS config namespaces](reference_ccs_config_namespaces.md) · [CCS AI config ns](reference_ccs_ai_chat_config_namespace.md) — AI-19 covers versioning/audit; no provider scope
- [CCS env topology](reference_ccs_env_topology.md) · [CCS3 frontend facts](reference_ccs3_frontend_facts.md) — dev ns on staging-th; members=/users
- [CCS global config gate](reference_ccs_global_config_permission_gate.md) · [CCS Go module broken](reference_ccs_go_module_path_broken.md) — view perm on sellsuki.user:""; MR !301 unmerged, gRPC no auth
- [rps dual mainline](reference_rps_dual_mainline.md) · [is_system_role trap](reference_rps_is_system_role_trap.md) · [kind must be prefixed](reference_rps_identity_kind_must_be_prefixed.md) · [ListAssignedRoles](reference_rps_list_assigned_roles_reverse_lookup.md) — rps facts
- [entity lib tenant kinds](reference_entity_lib_tenant_kinds.md) · [Permission generator churn](reference_permission_generator_nondeterministic.md) — IsActor allowlist; reorders per run
- [file-service Keto kind](reference_file_service_keto_subject_kind.md) — must match X-User-Kind
- [Messaging backend](reference_messaging_backend.md) · [repo traps](reference_messaging_backend_shared_repo_traps.md) — central OTP/SMS; .env tracked, no MR CI
- [Central audit log](project_central_audit_log.md) — PAT-2611 stdout→Loki→CCS UI
- [Audit Action closed enum](reference_audit_action_is_closed_enum.md) — use EntityRefs

## BOLA
- [Access model](project_bola_saas_access_model.md) · [auth mode](project_bola_auth_mode_deployment.md) · [RBAC keto-direct](project_bola_rbac_keto_direct.md) · [ops=CCS1](project_bola_ops_visibility_ccs1.md) · [Kratos SSO staging](project_bola_kratos_sso_staging.md) — CCS3=org, BOLA=workspace; SaaS=kratos+keto; outside AMS
- [Deploy topology](project_bola_deploy_topology.md) · [Kratos deploy gap](project_bola_saas_kratos_deploy_gap.md) · [values in repo](project_bola_deploy_values_in_repo.md) · [migrations on boot](project_bola_migrations_jsonb.md) — charts, secrets, crashloops
- [Contact profile](project_bola_contact_profile_model.md) · [contacts upsert](reference_bola_contacts_upsert_api.md) · [is_enabled mismatch](project_bola_is_enabled_int_bool_mismatch.md) · [segment export](project_segment_export_static_snapshot.md) — data model facts
- [Workspace scoping bugs](project_bola_workspace_scoping_bugs.md) — 65 confirmed leaks
- [Reply-token epic](project_bola_reply_token_epic.md) · [chatbot personalization](project_bola_ai_chatbot_personalization.md) · [APM webhook](project_bola_apm_webhook_design.md) · [APM scheduled+batch](project_apm_scheduled_batch_epic.md) — feature state
- [Staging Loki](reference_bola_staging_loki.md) · [BOLA-293 chain](project_bola293_chain_state.md) · [FB page dev-mode](project_fb_page_dev_mode_gate.md) — ops facts

## OC2Plus
- [OC-4207 LINE optional](project_oc4207_line_optional_design.md) — empty string = no LINE
- [OC epic triage](project_oc_epic_backlog_triage.md) — ~127 non-Done epics; 26 real
- [OC×BOLA boundary](project_oc_bola_domain_boundary.md) — LIFF register = OC2Plus
- [Member frontend](reference_oc2plus_member_frontend.md) · [member-api test login](reference_oc2plus_member_api_test_mode_login.md) — LIFF app; local session without LINE
- [OC-4267 standalone](project_oc4267_standalone_no_qms.md) — no QMS dependency
- [API-key gap](project_oc2plus_3rdparty_apikey_gap.md) · [OC-2275 audit](project_oc2275_audit_actionplan.md) · [API-key local run](project_oc2plus_apikey_local_run.md) · [TEST_KEY prod gate](project_oc2plus_test_key_production_gate.md) · [keyring vs OC-2275](project_sellsuki_keyring_vs_oc2275.md) — OC-2275 cluster
- [= company not store](reference_oc2plus_company_not_store.md) — store_id=0 throughout
- [Primary-invariant](project_oc2plus_primary_invariant_pattern.md) · [consent model](project_oc2plus_consent_enforcement_model.md) — patterns
- [Schema external](reference_oc2plus_schema_lives_in_external_repo.md) — DDL → member-api/migrations
- [OC-4362 claim cluster](project_oc4362_claim_cluster_gaps.md) · [OC-4464 OCR vendor](project_oc4464_ocr_vendor_decision.md) — Sprint 128; iApp, พ.ศ. trap
- [Customer App program](project_customer_app_program.md) · [auth plan](project_oc2plus_customer_app_auth_plan.md) — 5 epics; OC-4344 self-build
- [Invite→app chain](project_invite_multiapp_chain.md) — PAT-2553 return_to
- [Loyalty point cluster](project_loyalty_point_cluster.md) · [contract sheet](project_loyalty_canonical_contract.md) — OC-4413 = SoT

## Patona / OMS / QMS / SukiPay
- [Akita/Patona strategy](project_akita_patona_migration_strategy.md) — Patona = Akita migration target
- [OMS2 gaps](project_oms2_plan_gaps_2026q3.md) · [Decouple decision](project_oms2_decouple_decision.md) — PAT-2540
- [MS-687 reserve](project_ms687_reserve_needs_company_location.md) — needs company location
- [QMS CCS2 reframe](project_qms_ui.md) · [Quota ≠ gate](project_quota_not_feature_gate.md) · [no allow/deny RPC](reference_quota_no_allow_deny_rpc.md) · [Plan anchor](project_plan_capability_quota_anchor.md) — quota=metering (pre-AI-196); Commercial Plan = anchor
- [SukiPay audit table](project_sukipay_audit_log.md) · [void rename](project_sukipay_void_rename.md) · [refund cluster](project_sukipay_refund_cluster.md) · [offline payment](project_sukipay_offline_payment.md) — SukiPay facts
- [User pain evidence gap](project_user_pain_evidence_gap.md) — no real user research exists
- [Product KB](project_sellsuki_product_kb.md) — docs/product-kb/; verified/asserted/GAP tags
- [Bundle product](project_bundle_in_catalog.md) · [PIS frontend local](project_pis_frontend_local_testing.md) · [Provider FE/BE routing](project_provider_frontend_backend_routing.md) — misc service facts
- [ssk-* DS docs](reference_ssk_components_docs.md) · [DS 1.0 beta gotchas](reference_ds_1_0_beta_gotchas.md) — isCustomElement required; pin exact

## AI Chat Platform
- [Platform plan](project_ai_chat_platform_plan.md) — CTO 7-service; insurance pilot
- [Arch artifact](reference_ai_platform_architecture_artifact.md) — docs/ai-platform-architecture.md
- [Deploy gating](project_ai_platform_deploy_gating.md) — CI_JOB_ENABLE + k8s secret
- [Sprint 2-4 run](project_ai_sprint234_autonomous_run.md) · [merge order](project_ai_chatcore_merge_order.md) · [merge topology risk](project_ai_merge_topology_risk.md) — branches without MRs; codex baselines CLOSE not merge
- [FE design](reference_ai_chat_frontend_design.md) · [MVP integration](project_ai_mvp_integration.md) — DesignSync; works to token_unavailable
- [Board stale cards](reference_ai_board_stale_cards.md) · [In Review = merged unverified](reference_ai_board_in_review_means_merged_unverified.md) — never trust AI board status
- [ai-agent stateless](project_ai_agent_stateless.md) · [ai-agent CI gaps](reference_ai_agent_ci_gaps.md) — config on request; code_analyse always red
- [chat-core = admin BFF](reference_chatcore_is_the_admin_bff.md) · [role bootstrap](project_chatcore_role_bootstrap_and_eastwest_auth.md) · [unprefixed kind](reference_chatcore_unprefixed_chat_workspace.md) · [company list derived](project_chatcore_company_list_derived_not_asked.md) — chat-core facts
- [migrations 0066](reference_chatcore_migrations_break_at_0066.md) · [route tests timeout](reference_chatcore_route_tests_timeout_under_load.md) · [CI gaps](reference_chatcore_ci_gaps.md) — chat-core traps
- [rag-core dual embedding](project_ragcore_dual_embedding_paths.md) · [visibility tiers](reference_rag_core_visibility_tiers.md) · [KB 3 blockers](reference_kb_entries_three_blockers.md) — rag-core facts
- [Conversation Intelligence](project_ai_conversation_intelligence.md) — Case/Checkpoint/Memory-lanes
- [Gap sweep 2026-08](project_ai_backlog_gap_sweep_202608.md) — E12 ปิดครบ; AI-150 ติด decision
- [SLA ladder state](project_sla_ladder_engine_state.md) — MR order ai-agent !4 → chat-core !18 → FE !4
- [Flag without enforcement](reference_flag_without_enforcement.md) — ~40 routes unguarded
- [Admin port↔backend map](project_ai_admin_port_backend_map.md) · [Backend w/o FE invisible](reference_backend_without_frontend_is_invisible.md) — blocker is a seam; GAP markers one-directional
- [AI-150 members read-only](project_ai150_members_read_only.md) — CCS owns grants
- [E8 remaining blockers](project_e8_remaining_blockers.md) — verified 2026-08-30
- [Placeholder cards review](project_ai_placeholder_cards_review.md) — DEC-1..6 เคาะแล้ว 2026-09-01 (D10/OMS ยืน, token mode ตัด); E9 rescued → ai_chat_data_pipeline
