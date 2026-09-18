# Memory Index

Titles are short on purpose — each file's own `description:` carries the detail. Open the file before relying on it.

## User & how-to-work feedback
- [Thai](user_language_thai.md) · [Act don't list](feedback_decisive_deep_execution.md) · [Self-explaining UX](feedback_selfexplaining_ux.md) · [Subagent tiering](feedback_subagent_model_tiering.md)
- [Card = user story](feedback_card_user_story_flows.md) · [No mid-sprint scope](feedback_no_scope_change_in_sprint.md) · [QA guide](feedback_qa_reproduce_guide.md) · [Cite file:line](feedback_ground_claims_file_line.md)
- [Reproduce first](feedback_reproduce_the_number_before_asking.md) · [Verify as user](feedback_verify_as_the_user_sees_it.md) · [Check the norm](feedback_check_the_norm_before_calling_it_broken.md)
- [Grep the service, not the spec](feedback_grep_the_spec_is_not_grep_the_service.md)
- [Search before GAP](feedback_search_before_declaring_gap.md) · [Verify absence](feedback_verify_absence_claims.md) · [head = sampling](feedback_head_on_grep_is_sampling_not_verification.md) · [Whole stack](feedback_search_the_whole_stack_not_one_layer.md)
- [PMM voice](feedback_product_marketing_voice.md) · [Design fidelity](feedback_design_handoff_full_fidelity.md) · [Fix reaches all](feedback_fix_must_reach_everyone.md) · [Central ≠ caller](feedback_central_service_no_caller_domain.md) · [Entity pure](feedback_entity_pure_zod_dto.md) · [Report, don't edit](feedback_report_wrong_cards_dont_edit.md)
- [Codex ทำงานคู่ขนาน](feedback_user_runs_codex_in_parallel.md)
- [Commit+push ระหว่างทาง](feedback_commit_push_along_the_way.md)
- [No develop→main promotion MRs](feedback_no_develop_to_main_promotion_mrs.md) — main=staging; **5 ครั้ง ไม่ใช่ 2** (CCS3 + OC2Plus 3 repo); OC2Plus ก็ dual mainline
- [Parallel git safety](feedback_parallel_sessions_git_safety.md) · [List MRs first](feedback_list_open_mrs_before_opening_one.md) · [OC↔PAT boards](feedback_oc_pat_board_ownership_rule.md) · [OC → develop](feedback_oc2plus_merge_to_develop.md) · [PIS FF push](feedback_ff_only_force_push_ok.md)

## Personal / machine / infra
- [Merchant portal](project_merchant_portal.md) · [Ch.Erawan](project_ch_erawan_next.md) · [CATS ATS](reference_cats_ats_system.md) · [NAS DS1](reference_nas_ds1.md) · [Helio](project_helio.md) · [SecondBrain](reference_secondbrain_vault.md) · [Control Tower](project_control_tower.md)
- [OC2Plus dev URL + 3 บริษัท](reference_oc2plus_dev_urls_and_multicompany_grant.md) · [Env URLs](reference_env_urls.md) · [Monorepo remotes](reference_monorepo_no_origin.md) · [Mainline ≠ main](project_monorepo_mainline_is_not_main.md)
- [สายจริงของแต่ละ repo + 6 ตัวที่ develop เป็นป้ายเปล่า](reference_real_mainline_per_repo.md) — default_branch=main ทั้ง 55 repo เชื่อไม่ได้
- [OC2Plus stack traps](reference_oc2plus_local_stack_recovery_traps.md) · [Stale branches per repo](reference_local_stack_stale_branches_per_repo.md) · [Stray dev server](reference_stray_claude_dev_server_squats_port.md) · [Stale postgres pod](reference_datastore_stale_postgres_pod.md)
- [Branch เยอะ = เศษ worktree ไม่ใช่งานค้าง](reference_oc2plus_branch_sprawl_is_worktree_debris.md) — สูตรเคลียร์ 5 repo + กับดัก git cherry/dev server
- [สลับ branch แล้ว .env หาย](reference_branch_switch_can_lose_an_untracked_env.md) — messaging ตายด้วย "lookup port=5432"; กู้จาก commit 1482535
- [ย้าย branch ใต้ dev server ที่รันอยู่](reference_moving_a_branch_under_a_running_dev_server.md) — vite เสิร์ฟ .ts ดิบ / 404; เช็ค cwd ก่อนลบ
- [ฟีเจอร์ใหม่ไม่โผล่บน local](reference_local_stack_new_feature_invisible_two_causes.md) — เมนูหายเพราะ permission (เงียบ) + list 500 เพราะ internal key
- [overmind ตาย = session ค้าง](reference_overmind_dead_processes_are_a_wedged_session.md) — quit+rm sock+start ใหม่; และอย่ารัน service มือในโฟลเดอร์ที่ overmind ดูอยู่
- [Overmind restart](project_overmind_restart_quirk.md) · [bola overmind sock](reference_local_bola_own_overmind_socket.md) · [Caddy host net](reference_caddy_host_networking_gotcha.md) · [Browser surfaces](reference_browser_surfaces_this_workspace.md)
- [brew simdjson breaks node](reference_brew_simdjson_breaks_homebrew_node.md)
- [pre-push กัน credential ติดใน 56 repo](reference_prepush_credential_hook_installed.md) — push ถูกบล็อก = rotate แล้วเอาออก ห้าม --no-verify
- [rtk rewrites cmds](reference_rtk_git_output_filtering.md) · [Classifier blocks](reference_harness_classifier_blocks_secrets_and_mutations.md) · [Outline VPN](reference_outline_mcp_vpn_blocker.md) · [Agent resume](reference_background_agent_resume_patterns.md) · [codegraph projectPath](reference_codegraph_context_needs_projectpath.md)
- [CI history/DNS ≠ deploy status](reference_ci_history_and_dns_are_not_deploy_status.md) — ถาม kubectl; staging-th context พร้อมใช้
- [registry เต็ม 300 GiB = deploy ถูก skip เงียบ ๆ](reference_fountain_registry_quota_full.md) — test เขียว build แดง deploy skipped; MR เขียวไม่ได้แปลว่าของขึ้น
- [dev-th access](reference_dev_th_cluster_access.md) · [Teleport kills dev-th](reference_teleport_session_kills_devth_access.md)

## Jira
- [BOLA proj](reference_bola_jira_project.md) · [OC2Plus proj](reference_oc2plus_jira_project.md) · [Patona proj](reference_pat_jira_project.md) · [Sprint ids](reference_jira_sprint_ids_not_contiguous.md) · [PAT sprints](reference_pat_board_sprints.md) · [PAT epics unwired](project_pat_epic_links_unwired.md)
- [MCP crosses sessions](reference_jira_mcp_crosses_responses_between_sessions.md) · [MCP quirks](reference_jira_mcp_search_quirks.md) · [ADF + Thai mangling](reference_jira_editissue_adf_breakage.md) · [No local fallback](reference_no_local_jira_fallback.md)
- [Writes 403 mid-session](reference_jira_mcp_writes_403_midsession.md) — "app is not installed" while reads still work; retrying never clears it

## Git / CI gotchas
- [CRLF .vue reflow](reference_crlf_vue_files_reflow_on_text_rewrite.md) · [FF merge reverts](reference_fast_forward_merge_silently_reverts.md) · [Semantic break](reference_silent_semantic_merge_break.md) · [Parallel dup symbols](reference_parallel_sessions_duplicate_symbols.md) · [.bak drops comments](reference_bak_restore_drops_comments.md)
- [Cherry-pick looks like lost work](reference_cherry_pick_divergence_looks_like_lost_work.md) · [worktree rewinds](reference_worktree_remove_rewinds_main_checkout.md) · [Grep origin](reference_grep_stale_branch_not_origin.md) · [Shallow submodules](reference_submodules_are_shallow_clones.md) · [git @{u} false zero](reference_git_upstream_false_zero.md) · [Review bot reason](reference_review_bot_finding_right_reason_wrong.md)
- [Stale narrow fix MR reverts the broad one](reference_stale_narrow_fix_mr_reverts_the_broad_one.md)
- [Generated-file conflict → regenerate](reference_generated_file_merge_conflict_regenerate.md)
- [Stale conflict after force-push](reference_gitlab_stale_conflict_after_force_push.md)
- [glab merge 405 = pipeline ยังรัน](reference_glab_mr_merge_405_means_pipeline_running.md) — ไม่ใช่เรื่องสิทธิ์ รอ pipeline แล้วสั่งใหม่
- [Pipeline retry ships skipped deploys](reference_pipeline_retry_runs_skipped_deploy_jobs.md) · [Private Go module CI](reference_gitlab_private_go_module_ci.md) · [Review-bot targets](reference_gitlab_review_bot_targets.md) · [Dead staging runner](reference_dead_staging_runner_tag.md) · [glab ci stale](reference_glab_ci_status_stale_pipeline.md) · [glab --auto-merge ไม่รอ CI](reference_glab_auto_merge_does_not_wait.md) · [rules de-scope jobs](reference_gitlab_rules_silently_descope_jobs.md) · [Library skips SRE tpl](reference_shared_library_skips_sre_template.md) · [Coverage on DB-less](reference_coverage_gate_on_dbless_job.md)

## Test gotchas
- [Green count hides an uncollected suite](reference_green_count_hides_uncollected_suite.md) · [Sentry dual hub on skew](reference_sentry_dual_hub_on_version_skew.md)
- [Stub too permissive](reference_test_stub_more_permissive_than_service.md) · [testify default wins](reference_testify_permissive_default_wins.md) · [Timing concurrency](reference_timing_dependent_concurrency_tests.md) · [Turbo false green](reference_turbo_cache_crosssession_false_green.md) · [Lit/React SSR](reference_lit_react_node_condition_hollows_tests.md) · [DS testId](reference_ds_testid_is_a_property.md) · [Node 25 jsdom](reference_node25_localstorage_jsdom_conflict.md)

## Go / DB gotchas
- [Migration files ≠ applied schema](reference_migration_files_are_not_applied_schema.md)
- [envDefault localhost ปิดบัง config ที่หายไป](reference_envdefault_localhost_masks_missing_config.md) — กลายเป็น connection error แทน config error; ลบ default ทิ้ง อย่าแก้ค่า
- [goqu dialect blank import](reference_goqu_dialect_blank_import.md)
- [AutoMigrate 2nd boot](reference_gorm_pgx_libpq_automigrate.md) · [Updates drops false](reference_gorm_updates_drops_false.md) · [Lease/claim class](reference_lease_claim_ownership_bug_class.md) · [Partial-index ON CONFLICT](reference_pg_partial_index_onconflict_generic_plan.md) · [Ambiguous 404](reference_ambiguous_404_fail_open.md) · [Auth behind own guard](reference_auth_endpoint_behind_own_guard.md) · [Kafka silent publish](reference_kafka_silent_publish_failure.md)

## Identity / CCS / rps
- [Identity ≠ can sign in](reference_kratos_identity_exists_is_not_registered.md) · [Identity SPOF](reference_shared_identity_infra_singleton.md) · [Local identity loop](reference_local_identity_hardcode_loop.md) · [Local Kratos debug](reference_local_kratos_identity_debugging.md) · [Local i18n seed](reference_local_i18n_config_seeding.md)
- [CCS config ns](reference_ccs_config_namespaces.md) · [AI config ns](reference_ccs_ai_chat_config_namespace.md) · [CCS env topology](reference_ccs_env_topology.md) · [CCS3 FE facts](reference_ccs3_frontend_facts.md) · [Global config gate](reference_ccs_global_config_permission_gate.md) · [CCS Go module broken](reference_ccs_go_module_path_broken.md)
- [CCS deployment values are in the repo](reference_ccs_deployment_values_live_in_the_repo.md) — `deployment/values-*.yml`; ports there are each service's OWN defaults (rps HTTP=80, gRPC=50051), never the monorepo Procfile's
- [CCS3 QA: tell them before you change these 3 things](project_ccs3_qa_change_notification_agreement.md) — table columns, Company Owner permissions, status/error shapes; plus the `user-list.*` data-testid contract
- [CCS config: userId = schema defaults, 200](reference_ccs_config_userid_silently_returns_schema_defaults.md)
- [Keto ≠ rps catalog](reference_keto_and_rps_catalog_disagree.md) · [grant permission ให้ role](reference_rps_grant_permission_to_role.md)
- [Invite accept burns the code on grant failure](reference_invite_accept_burns_the_code_on_grant_failure.md) — usage committed to PG before Keto grants; fixed by compensation; CCS `INVITATION_API_BASE_URL` still unset on staging
- [proto ของ rps ถูกก็อปไว้ 9 repo](reference_rps_proto_is_vendored_per_consumer.md) — แก้ที่ rps ไม่ไหลไปไหนเอง; มีแต่ CCS ที่ใช้ invitation RPC; rps ต้อง deploy ก่อนเสมอ
- [rps internal endpoints](reference_rps_internal_endpoints_confirmed.md) · [ListRoles paging](reference_rps_listroles_pointer_pagination.md) · [Keto staging lookup](reference_keto_staging_permission_lookup.md) · [Presets at creation only](project_ccs_role_presets_apply_only_at_creation.md) · [rps dual mainline](reference_rps_dual_mainline.md) · [is_system_role](reference_rps_is_system_role_trap.md) · [kind prefixed](reference_rps_identity_kind_must_be_prefixed.md) · [ListAssignedRoles](reference_rps_list_assigned_roles_reverse_lookup.md)
- [Permission code underscore rejected pre-query](reference_permission_code_underscore_rejected_before_query.md)
- [entity tenant kinds](reference_entity_lib_tenant_kinds.md) · [Perm generator churn](reference_permission_generator_nondeterministic.md) · [file-service Keto kind](reference_file_service_keto_subject_kind.md) · [Audit Action enum](reference_audit_action_is_closed_enum.md) · [Messaging backend](reference_messaging_backend.md) · [Messaging repo traps](reference_messaging_backend_shared_repo_traps.md) · [Central audit log](project_central_audit_log.md)

## BOLA
- [bola-dev มีอยู่จริงและ deploy เอง](reference_bola_dev_env_exists_and_is_live.md) — CI_JOB_ENABLE ตั้งที่ GitLab project variable; grep ในรีโปไม่เห็น อ่านไฟล์อย่างเดียวสรุปผิดสามรอบ
- [Manual staging gate](reference_manual_staging_gate_silent_drift.md) · [CCS→BOLA unwired](project_ccs_bola_provisioning_unwired.md) · [Deploy topology](project_bola_deploy_topology.md) · [values in repo](project_bola_deploy_values_in_repo.md) · [Kratos deploy gap](project_bola_saas_kratos_deploy_gap.md) · [migrations on boot](project_bola_migrations_jsonb.md) · [Staging Loki](reference_bola_staging_loki.md)
- [§6a invite via CCS](project_bola309_invite_via_ccs.md) · [§6a lane 2 grant](project_bola309_lane2_add_existing_member.md) · [Access model](project_bola_saas_access_model.md) · [auth mode](project_bola_auth_mode_deployment.md) · [RBAC keto-direct](project_bola_rbac_keto_direct.md) · [ops = CCS1](project_bola_ops_visibility_ccs1.md) · [Kratos SSO staging](project_bola_kratos_sso_staging.md)
- [Follower metadata ×2](reference_bola_follower_metadata_two_stores.md) · [Contact profile](project_bola_contact_profile_model.md) · [contacts upsert](reference_bola_contacts_upsert_api.md) · [INTEGER vs bool](project_bola_is_enabled_int_bool_mismatch.md) · [Segment export](project_segment_export_static_snapshot.md) · [Workspace scoping](project_bola_workspace_scoping_bugs.md)
- [BOLA-293 chain](project_bola293_chain_state.md) · [FB page dev-mode](project_fb_page_dev_mode_gate.md) · [Reply-token epic](project_bola_reply_token_epic.md) · [Chatbot personalization](project_bola_ai_chatbot_personalization.md) · [APM webhook](project_bola_apm_webhook_design.md) · [APM scheduled+batch](project_apm_scheduled_batch_epic.md)

## OC2Plus
- [LINE optional](project_oc4207_line_optional_design.md) · [Epic triage](project_oc_epic_backlog_triage.md) · [OC×BOLA boundary](project_oc_bola_domain_boundary.md) · [OC-4267 standalone](project_oc4267_standalone_no_qms.md) · [Company not store](reference_oc2plus_company_not_store.md) · [Tier per-company](project_oc2plus_tier_is_per_company_config.md)
- [BO FE red on develop](reference_oc2plus_backoffice_fe_red_on_develop.md)
- [BO FE: CI ข้ามเทสต์ทั้งหมด](reference_backoffice_fe_ci_skips_all_tests.md) — `UNIT_TEST_SCRIPT: echo "1 + 1"`; develop แดง 181 เทสต์ pipeline ยังเขียว
- [FE worktree → branch API](reference_oc2plus_backoffice_fe_local_against_branch_api.md)
- [OC-4086 editor ไม่มีจริง](reference_oc4086_rich_text_editor_not_in_monorepo.md) — Done แต่โค้ดไม่อยู่ใน monorepo อย่าไล่หา
- [e2e repo (proj 808)](reference_oc2plus_e2e_playwright_repo.md) · [SRE slots no `&&`](reference_sre_test_job_slots_cannot_chain.md) · [Kit needs pnpm](reference_frontend_kit_consumption_needs_pnpm.md) · [Kit state](reference_frontend_kit_state.md) · [CI outage 2026-09](reference_oc2plus_ci_outage_2026_09.md)
- [Unset KAFKA_TOPIC_* = 500 cannot_publish_message](reference_unset_kafka_topic_is_a_500.md)
- [file-service grant is per-company](reference_file_service_grant_is_per_company.md)
- [Register minted no session](reference_oc2plus_register_did_not_mint_a_session.md)
- [Naive timestamp = host TZ](reference_naive_timestamp_columns_shift_by_host_tz.md)
- [Member page registry (backend-owned)](reference_oc2plus_member_page_registry.md)
- [React route flip needs public allow-list](reference_react_route_owner_flip_needs_public_allowlist.md)
- [gender enum typo in backoffice spec](reference_oc2plus_gender_enum_typo_in_backoffice_spec.md)
- [Local QA session for the member app](reference_oc2plus_member_app_local_qa_session.md)
- [OC-4356 News dev blockers](project_oc4356_news_cms_dev_blockers.md) — 017+018 apply แล้ว + news.manage granted 4536 roles (2026-09-13); เหลือ 019 coupon + member_tier และ staging/prod
- [OC-4526 กระเป๋าคูปอง](project_oc4526_coupon_wallet.md) — พนักงานสแกนถึงตัดคูปอง, expired คำนวณตอนอ่าน; ฝั่ง backoffice ว่างเปล่า → เปิด OC-4546/4547 แล้ว
- [OC-4530 สแกนที่เคาน์เตอร์](project_oc4530_counter_scan_reanalysis.md)
- [OC-4363 กรอกรหัส 409/410](project_oc4363_code_redemption_state.md)
- [OC-4529 บัตรสมาชิก QR](project_oc4529_member_card_token.md)
- [UI polish rules](project_oc2plus_member_ui_polish_conventions.md) — สีแบรนด์=กดได้, ป้ายกริด=caption, rail ต้อง flex
- [Member design v2](project_oc2plus_member_app_design_v2.md) · [React migration](project_oc2plus_member_react_migration.md) · [--c-* vars dead](reference_oc2plus_member_c_vars_are_dead.md) · [Member frontend](reference_oc2plus_member_frontend.md) · [member-api test login](reference_oc2plus_member_api_test_mode_login.md) · [codegen is Go](reference_oc2plus_backoffice_codegen_is_go.md) · [Schema external](reference_oc2plus_schema_lives_in_external_repo.md) · [CRM migrations by hand](project_oc2275_crm_migrations_run_by_hand.md) — สูตรรันมือ + dev/staging ตามทันแล้ว prod ยังไม่ได้ · [DaisyUI collision](reference_daisyui_progress_class_collision.md)
- [OC-2275 blocked](project_oc2275_remaining_blocked_on_decisions.md) · [API-key gap](project_oc2plus_3rdparty_apikey_gap.md) · [OC-2275 audit](project_oc2275_audit_actionplan.md) · [API-key local run](project_oc2plus_apikey_local_run.md) · [TEST_KEY prod gate](project_oc2plus_test_key_production_gate.md) · [keyring vs 2275](project_sellsuki_keyring_vs_oc2275.md) · [Prod v2 unverified](project_oc2plus_prod_v2_apikey_unverified.md)
- [Primary invariant](project_oc2plus_primary_invariant_pattern.md) · [Consent model](project_oc2plus_consent_enforcement_model.md) · [Loyalty cluster](project_loyalty_point_cluster.md) · [Contract sheet](project_loyalty_canonical_contract.md) · [OC-4415 base rate](project_oc4415_base_rate_state.md) · [pointclaim perm missing](project_pointclaim_permission_missing_from_owner_preset.md)
- [Engine ไม่มีคนเรียก](project_award_engine_has_no_callers.md) · [ซ้อนแคมเปญ = เอาใบดีที่สุด](project_loyalty_overlap_best_single_campaign.md) — approve จ่าย base rate ล้วน, adapter To Do ทุกใบ
- [OC-3559 tier state](project_oc3559_member_tier_state.md)
- [OC-4560 slug แก้ได้](project_oc4560_editable_member_app_slug.md) — เก็บบน binding ไม่ใช่ company.Code; upsert ห้ามขยับ slug
- [OC-4362 claim gaps](project_oc4362_claim_cluster_gaps.md) · [OCR = VLM](project_oc4464_ocr_vendor_decision.md) · [Approve + admin edit](project_oc4362_approve_and_admin_edit.md)
- [BFF reads direct, not proxy](project_oc2plus_customer_bff_reads_direct_not_proxy.md)
- [consent binding ไม่มีคนเขียน](reference_oc2plus_consent_binding_has_no_writer.md) — ตาราง `consent` ไม่มี INSERT ที่ไหนเลย (OC-4089 ยัง To Do); key ด้วย OA ทำให้เส้นเว็บพัง → OC-4545
- [OC-4551 checklist ก่อนเปิดใช้](project_oc4551_readiness_checklist.md) — ส่งครบทั้ง BE+FE แล้ว; unknown ปิดเกท
- [OC-4089 หน้าผูก consent (mock)](project_oc4089_consent_binding_page.md) — enforcement อยู่ที่ option; company_consent รับแค่ pdpa/tos
- [OTP session = 403 consent](reference_oc2plus_otp_session_fails_3rdparty_consent.md)
- [Customer App program](project_customer_app_program.md) · [Auth plan](project_oc2plus_customer_app_auth_plan.md) · [Web-OTP minter](project_oc4348_web_otp_session_minter.md) · [Invite→app chain](project_invite_multiapp_chain.md) · [BOLA binding via CCS](project_bola_binding_never_worked_via_ccs.md) · [OC-4511..4514 UX](project_oc4511_4514_ux_cluster.md) · [LIFF shell = entry](project_oc2plus_liff_shell_is_the_line_entry.md)
- [OC-4523 + BOLA-328 LINE login](project_oc4523_line_login_cards.md)

## Patona / OMS / QMS / SukiPay
- [Akita strategy](project_akita_patona_migration_strategy.md) · [OMS2 gaps](project_oms2_plan_gaps_2026q3.md) · [Decouple decision](project_oms2_decouple_decision.md) · [MS-687 reserve](project_ms687_reserve_needs_company_location.md)
- [QMS: 21 ใบ code review มีโค้ดครบ แต่ค้าง 3 MR ตั้งแต่ ก.ค. + การ์ดระบุรีโปผิด](project_qms_cards_code_lives_elsewhere.md)
- [QMS CCS2](project_qms_ui.md) · [Quota ≠ gate](project_quota_not_feature_gate.md) · [No allow/deny RPC](reference_quota_no_allow_deny_rpc.md) · [Plan anchor](project_plan_capability_quota_anchor.md)
- [SukiPay audit](project_sukipay_audit_log.md) · [void rename](project_sukipay_void_rename.md) · [refund cluster](project_sukipay_refund_cluster.md) · [offline payment](project_sukipay_offline_payment.md)
- [User pain gap](project_user_pain_evidence_gap.md) · [Product KB](project_sellsuki_product_kb.md) · [Bundle in catalog](project_bundle_in_catalog.md) · [PIS FE + embeds](project_pis_frontend_local_testing.md) · [Provider routing](project_provider_frontend_backend_routing.md)
- [DS invented props](reference_ds_invented_props_render_nothing.md) · [ssk-* docs](reference_ssk_components_docs.md) · [DS 1.0 beta](reference_ds_1_0_beta_gotchas.md) · [DS = 94% bundle](reference_ds_bundle_dominates_and_cannot_treeshake.md) · [DS inputs slow](reference_ds_inputs_expensive_on_first_paint.md) · [i18next mutates JSON](reference_i18next_mutates_the_json_you_import.md)
- [Svelte `$:` self-dependency freezes the tab](reference_svelte_reactive_self_dependency_freezes_tab.md)

## AI Chat Platform
- [Platform plan](project_ai_chat_platform_plan.md) · [Arch artifact](reference_ai_platform_architecture_artifact.md) · [E0 structure](project_ai_chatsystem_e0_structure.md) · [Deploy gating](project_ai_platform_deploy_gating.md) · [FE design](reference_ai_chat_frontend_design.md) · [MVP integration](project_ai_mvp_integration.md)
- [Chat platform is vertical-neutral](project_chat_platform_is_vertical_neutral.md)
- [Sprint 2-4 run](project_ai_sprint234_autonomous_run.md) · [Merge order](project_ai_chatcore_merge_order.md) · [Merge topology risk](project_ai_merge_topology_risk.md) · [SLA ladder](project_sla_ladder_engine_state.md)
- [Board stale](reference_ai_board_stale_cards.md) · [In Review = merged](reference_ai_board_in_review_means_merged_unverified.md) · [Placeholder cards](project_ai_placeholder_cards_review.md) · [Gap sweep 2026-08](project_ai_backlog_gap_sweep_202608.md) · [E8 blockers](project_e8_remaining_blockers.md) · [Completeness audit](reference_ai_chatbot_completeness_audit.md)
- [ai-agent stateless](project_ai_agent_stateless.md) · [ai-agent CI gaps](reference_ai_agent_ci_gaps.md) · [Flag w/o enforcement](reference_flag_without_enforcement.md) · [PAT-2658 collision](project_pat2658_reference_collision.md)
- [chat-core = admin BFF](reference_chatcore_is_the_admin_bff.md) · [Role bootstrap](project_chatcore_role_bootstrap_and_eastwest_auth.md) · [Unprefixed kind](reference_chatcore_unprefixed_chat_workspace.md) · [Company list derived](project_chatcore_company_list_derived_not_asked.md) · [migrations 0066](reference_chatcore_migrations_break_at_0066.md) · [Route tests timeout](reference_chatcore_route_tests_timeout_under_load.md) · [route/workspace flaky](reference_chatcore_route_workspace_flaky_under_load.md) · [Zero-value panic](reference_chatcore_zero_value_usecase_panics.md) · [chat-core CI gaps](reference_chatcore_ci_gaps.md)
- [rag-core dual embed](project_ragcore_dual_embedding_paths.md) · [Visibility tiers](reference_rag_core_visibility_tiers.md) · [KB 3 blockers](reference_kb_entries_three_blockers.md) · [Conversation Intel](project_ai_conversation_intelligence.md)
- [Admin port↔backend](project_ai_admin_port_backend_map.md) · [Backend w/o FE](reference_backend_without_frontend_is_invisible.md) · [AI-150 read-only](project_ai150_members_read_only.md) · [AI-119 deferred](project_ai119_push_deferred.md) · [AI-115 backend only](project_ai115_conversation_goal_backend_only.md) · [AI-146 not started](project_ai146_onboarding_wizard_not_started.md) · [case_type home](project_case_type_setting_has_no_home.md) · [Provider create = CCS1](project_provider_create_lives_in_ccs1.md)
- [preferred_language = th](project_preferred_language_is_constant_th.md) · [Fact vocab resolved](project_fact_vocabulary_collision.md) · [AI-96 catalog](project_ai96_template_catalog_reality.md) · [AI-16 field set](project_ai16_field_set_admin_configurable.md) · [AI-125 OAuth](project_ai125_oauth_long_lived_server_side.md) · [AI-33 WebSocket](project_ai33_websocket_is_required.md) · [F08 schema vs values](project_f08_fact_schema_versus_values.md)
- [Stuck job holds group](reference_stuck_ci_job_holds_resource_group.md)
- [FB page must be subscribed to the app](reference_fb_page_must_be_subscribed_to_app.md)
- [ถาม Meta ตรง ๆ ว่าทำไมเพจเงียบ](reference_fb_page_delivery_diagnosis_endpoint.md) — delivery-diagnosis endpoint; พิสูจน์ว่า config ของเป็ดน้อยเหมือนเพจที่ใช้ได้เป๊ะ
- [ack 200 แล้วทิ้ง = ดูเหมือนเขาไม่ส่งมา](reference_webhook_ack200_and_drop_looks_like_no_delivery.md) — อ่านแค่ messaging[0] ทิ้งทั้ง batch; instrument จุดปฏิเสธก่อนตั้งสมมติฐานเรื่องผู้ส่ง
- [AI Chat ไม่มีอยู่บน staging](reference_ai_chat_has_no_staging_deployment.md) — chat-core ไม่เคย deploy, chat module ปิด ⇒ e2e staging รันไม่ได้และไม่เคยรัน
- [Procfile.messaging ปิด chat module](reference_procfile_messaging_disables_chat_module.md) — /webhook/fb 404 เงียบ ๆ ทั้งที่ port ดูปกติ; พิสูจน์ endpoint ตัวเองก่อนโทษ Meta
- [4 คีย์ env ที่ไม่มีใน .env เลย ทำ AI-chat local พัง](reference_ai_chat_local_env_keys_missing.md) — ตารางคีย์+ค่า; 404/503 ที่ชี้ผิดชั้นทุกอัน
- [chat-core ไม่มี ORY_KRATOS_PUBLIC_URL = คอนโซลตาย](reference_chatcore_missing_kratos_url_503.md) — /v1/me/* 503 แต่ curl ไม่มี cookie ยังตอบ 401 สวย ๆ
- [Fiber ctx strings ทำ label พัง](reference_fiber_ctx_strings_corrupt_prometheus_labels.md) — ต้อง strings.Clone ไม่งั้น /metrics 500 ทั้งก้อน เมตริกหายหมด
