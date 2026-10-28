# Memory Index

Titles are 1–3 words on purpose — each file's `description:` carries the real detail. Always open the file before relying on it. Domain detail lives in the sub-indexes at the bottom; open the one for the topic at hand.

## Feedback & how to work
- [Thai](user_language_thai.md) · [Act](feedback_decisive_deep_execution.md) · [Self-explain](feedback_selfexplaining_ux.md) · [Subagents](feedback_subagent_model_tiering.md) · [PMM voice](feedback_product_marketing_voice.md) · [Design fidelity](feedback_design_handoff_full_fidelity.md) · [Entity pure](feedback_entity_pure_zod_dto.md)
- [Card=story](feedback_card_user_story_flows.md) · [No mid-sprint](feedback_no_scope_change_in_sprint.md) · [QA guide](feedback_qa_reproduce_guide.md) · [Cite line](feedback_ground_claims_file_line.md) · [Report not edit](feedback_report_wrong_cards_dont_edit.md) · [Fix reaches all](feedback_fix_must_reach_everyone.md) · [Central≠caller](feedback_central_service_no_caller_domain.md)
- [Reproduce](feedback_reproduce_the_number_before_asking.md) · [As user](feedback_verify_as_the_user_sees_it.md) · [Norm first](feedback_check_the_norm_before_calling_it_broken.md) · [Grep service](feedback_grep_the_spec_is_not_grep_the_service.md) · [Before GAP](feedback_search_before_declaring_gap.md) · [Absence](feedback_verify_absence_claims.md) · [head=sample](feedback_head_on_grep_is_sampling_not_verification.md) · [All layers](feedback_search_the_whole_stack_not_one_layer.md)
- [No bug card](feedback_no_bug_card_for_in_sprint_fixes.md) · [Codex](feedback_user_runs_codex_in_parallel.md) · [Push often](feedback_commit_push_along_the_way.md) · [Git flow](feedback_git_flow_main_develop_main.md) (main→develop test→same branch to main) · [No promo MR](feedback_no_develop_to_main_promotion_mrs.md) · [Git safety](feedback_parallel_sessions_git_safety.md) · [List MRs](feedback_list_open_mrs_before_opening_one.md) · [OC↔PAT](feedback_oc_pat_board_ownership_rule.md) · [OC→develop](feedback_oc2plus_merge_to_develop.md) (partly superseded) · [PIS FF](feedback_ff_only_force_push_ok.md) · [No tiny cards](feedback_no_tiny_cards_bundle_as_ac.md)
- [Generator no rmtree](feedback_generator_must_not_wipe_shared_dirs.md) — write only marker files
- [Nested po-lead](feedback_nested_po_lead_loses_drafts_write_to_files.md) — spawn writers from main session, write to files
- [gemini-2.5-flash only](feedback_gemini_25_flash_is_the_kb_answer_model.md) — KB answer model fixed 10-07
- [Tests from card](feedback_tests_derive_from_the_card_not_the_code.md) — card = SSOT, never fit expected to code
- [Standing approval staging](feedback_standing_approval_merge_deploy_staging_after_gates.md) — gates green → merge/deploy staging myself · prod still ask
- [Comment card](feedback_comment_jira_card_when_loop_closes.md) — Jira comment + status follows MR
- [Urgent thread](feedback_keep_urgent_thread_visible.md) — don't bury live damage under a new topic
- [Sweep Codex branches](feedback_sweep_codex_branches_and_ship_whole.md) — Codex release branch is a superset; its BLOCKED ≠ PO blocker
- [Ask with full context](feedback_ask_decisions_with_full_context_not_bare_lists.md) — what/why/options+cost/recommendation
- [Check env yourself](feedback_check_read_only_env_facts_yourself_user_does_not_know.md) — port-forward rps + grpcurl, don't ask
- [Read Job logs first](feedback_read_job_logs_before_deleting_the_job.md) — save logs before deleting a Job
- [Morning retries in waves](feedback_morning_retries_in_waves_runner_cluster_is_12_nodes.md) — ≤6–8 jobs per wave, 12-node runner
- [Sprint sweep = deploy](feedback_sprint_sweep_board_column_means_deploy_not_ac_proof.md) — column means deployed, not AC proven
- [Mock review](feedback_mock_mode_ui_review_check_real_adapter_before_calling_defect.md) — check real adapter before calling a defect
- [rtk cat strips comments](feedback_rtk_cat_strips_comments_never_use_cat_to_rewrite_files.md) — write files with python bytes
- [rtk proxy to compare](feedback_compare_command_outputs_with_rtk_proxy.md) — rtk truncates output; read raw
- [Vault/prod: hand over commands](user_has_vault_and_prod_access_agent_hands_over_commands.md) — nunt has Vault+prod access

## Git / branches
- [Stacked promotion wave](reference_stacked_promotion_wave_develop_to_main.md) — chain-merge in scratch worktree, user runs merge script
- [Auto mode blocks MR create](reference_auto_mode_blocks_glab_mr_create.md) — prepare JSON + create-mrs.sh · project ids inside
- [OC2Plus develop↔main diverged](reference_oc2plus_backoffice_develop_main_diverged_2026_10_11.md) — OC-4736 one-time reunion
- [rps/mgmt-backend lines](reference_rps_and_management_backend_develop_main_diverged.md) — rps not diverged, mgmt-backend is
- [Night = staging-th only](reference_night_window_is_staging_th_only_production_runners_run.md) — prod runners still run
- [First CI run of old specs](reference_first_ci_run_of_never_run_playwright_specs_finds_real_bugs_and_axe_baseline_drift.md) — real bugs + axe drift · never rebase pushed

## Projects / state
- [nunt-mode profile](project_nunt_mode_agent_profile.md) — /my-mode, eval REVISE, not shared yet
- [Tiers + Decision Inbox](project_workflow_tiers_and_inbox_2026_10_13.md) — AC tiers A/B/C + question queue
- [OC-4739 file-service backfill](project_oc4739_file_service_access_backfill.md) — staging 14,749 companies · AC8 left
- [Oc2plus SaaS OC-4763](project_oc2plus_saas_plan_epic_oc4763_decisions_2026_10_16.md) — PO quota/billing decisions 10-16
- [System testing direction](project_system_testing_direction_2026_10_11.md) — invariants suite, seed-scenario, ratchet
- [Mahanakhon decisions](project_mahanakhon_proposal_vs_roadmap_decisions_2026_10_14.md) — CRM owns active/tier, per company
- [Playbook/Goal/Lead/KB cards](project_playbook_goal_lead_kb_cards_2026_10_24.md) — AI-312..327 + PO verdicts
- [Wave 1 state 10-25](project_wave1_playbook_goal_lead_kb_state_2026_10_25.md) — 8 cards pushed, no MR/verifier
- [AI-318 depends on model](project_ai318_interest_gate_depends_on_the_model_2026_10_25.md) — gemini fails, gpt-4o-mini passes · D-038
- [AI chat dev order 10-27](reference_ai_chat_dev_order_reconcile_2026_10_27.md) — 5 waves · both sprint cleanups applied (21=39/22=47/23=34/24=11) · Sprint 1-20 closed, 21 active via pane REST · staging chat module off
- [AI chat permission staging 10-27](project_ai_chat_staging_permission_state_2026_10_27.md) — Owner 3 generations · no usage/billing preset · switcher bug · dry-run numbers · fix plan · AI-252 develop-only
- [Unscheduled sweeps AI-351](project_chat_core_unscheduled_sweeps_ai351.md) — 9 chat-core sweeps had no CronJob · rolling-summary added, rest wait D-051
- [Messaging chat flip 10-27](reference_messaging_chat_module_staging_flip_2026_10_27.md) — off by !80 09:59, back on by !82 16:46 (deploy 275941) · develop untouched · VAULT_TOKEN ~11-23 · M1–M8 in docs/cards/AI-216.md

## Jira
- [Sweep via files](reference_jira_board_sweep_via_saved_results.md) · [BOLA](reference_bola_jira_project.md) · [OC2Plus](reference_oc2plus_jira_project.md) · [Patona](reference_pat_jira_project.md) · [Sprint ids](reference_jira_sprint_ids_not_contiguous.md) · [PAT sprints](reference_pat_board_sprints.md) · [PAT epics](project_pat_epic_links_unwired.md) · [Crosses sessions](reference_jira_mcp_crosses_responses_between_sessions.md) · [Quirks](reference_jira_mcp_search_quirks.md) · [ADF Thai](reference_jira_editissue_adf_breakage.md) · [No fallback](reference_no_local_jira_fallback.md) · [403 writes](reference_jira_mcp_writes_403_midsession.md) · [BOLA no In Review](reference_bola_jira_has_no_in_review_status_and_auto_mode_denies_merge.md)

## Tests
- [slog.Init once](reference_slog_init_once_silences_the_test_binary.md) · [ssk user-detail](reference_ssk_widget_user_detail_height_is_a_string.md) · [Robot CI no target](reference_sukispace_robot_ci_has_no_target_service.md) · [Robot var case](reference_robot_variables_are_case_insensitive_loop_var_overwrites_suite_var.md) · [BOLA PG tests](reference_bola_postgres_tests_need_disposable_container_and_two_dsn_forms.md)
- [Uncollected](reference_green_count_hides_uncollected_suite.md) · [Sentry dual](reference_sentry_dual_hub_on_version_skew.md) · [Stub loose](reference_test_stub_more_permissive_than_service.md) · [testify](reference_testify_permissive_default_wins.md) · [Timing](reference_timing_dependent_concurrency_tests.md) · [Turbo](reference_turbo_cache_crosssession_false_green.md) · [Lit SSR](reference_lit_react_node_condition_hollows_tests.md) · [DS testId](reference_ds_testid_is_a_property.md) · [jsdom 25](reference_node25_localstorage_jsdom_conflict.md)
- [chat-core DB tests -p 1](reference_chat_core_db_tests_need_p1.md) — parallel run → "File exists" on TRUNCATE
- [Scenario harness](reference_real_conversation_scenario_harness_lead_goal_step_2026_10_24.md) — turn-by-turn customer chat on local

## Go / DB / sandbox
- [ns octoplus](reference_oc2plus_cluster_namespace_is_octoplus.md) · [Files≠schema](reference_migration_files_are_not_applied_schema.md) · [envDefault](reference_envdefault_localhost_masks_missing_config.md) · [goqu](reference_goqu_dialect_blank_import.md) · [AutoMigrate](reference_gorm_pgx_libpq_automigrate.md) · [Drops false](reference_gorm_updates_drops_false.md) · [Lease](reference_lease_claim_ownership_bug_class.md) · [Partial idx](reference_pg_partial_index_onconflict_generic_plan.md) · [404 open](reference_ambiguous_404_fail_open.md) · [Own guard](reference_auth_endpoint_behind_own_guard.md) · [Kafka silent](reference_kafka_silent_publish_failure.md) · [Fiber ctx](reference_fiber_ctx_strings_corrupt_prometheus_labels.md)
- [pgx cached plan](reference_pgx_cached_plan_breaks_old_pods_after_add_column_migration.md) — old pods error after ADD COLUMN
- [Sandbox: DB tests skip](reference_sandbox_blocks_localhost_so_db_tests_skip_silently.md) · [Sandbox: Go CLI TLS](reference_sandbox_blocks_go_cli_tls_and_ssh.md) · [TMPDIR differs](reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash.md) — use scratchpad absolute path
- [Disk full → Docker dies](reference_disk_full_kills_docker_desktop_go_cache_is_the_first_thing_to_delete.md) — delete go cache first

## AI chat / KB
- [KB100 eval + RAG direction](project_kb100_eval_state_and_rag_direction.md) · [local kb_rag DB](reference_local_kb_rag_db_and_milvus.md) · [Drive MCP by ID](reference_google_drive_mcp_shared_files_by_id.md) · [poppler 25.03 Thai](reference_poppler_2503_breaks_thai_pdf_text.md) · [402 quota](reference_ai_agent_quota_exhausted_is_openrouter_402.md)
- [Staging = ns sellsuki](reference_ai_chat_staging_is_ns_sellsuki_and_how_to_release_there.md) · [Replay in pod](reference_staging_chat_core_replay_recipe_in_pod_wget.md) · [Inbound forward local](reference_chat_core_local_inbound_forward_for_live_tests.md) · [Connector 403](reference_connector_403_is_often_chat_core_token_mismatch.md)
- [Pane shadow DOM inputs](reference_ai_chat_console_pane_shadow_dom_inputs_need_js_setter.md) — set value via native setter, click inner button; pane records no fetch/console
- [Console local](reference_local_ai_chat_console_open_on_localhost_with_seed_identity.md) · [Console staging pane](reference_staging_ai_chat_console_drive_recipe.md) · [Console mock :5192](reference_ai_chat_admin_mock_preview_5192_no_login.md) · [Verifier preflight](reference_staging_verifier_preflight_tsh_and_pane_login.md) · [MR+Jira recipe](reference_ai_chat_mr_and_jira_landing_recipe_2026_10_25.md)
- [Agent Vault expiry](reference_ai_agent_staging_vault_token_expiry_crash_on_restart.md) · [Messaging Vault 403](reference_messaging_backend_staging_vault_token_403_makes_fb_webhook_500.md) · [503 = pg18 disk](reference_staging_chat_core_503_is_often_shared_postgres_pg18_disk_full.md)
- [Morning restarts = Keto](reference_staging_morning_boot_restarts_on_keto_are_expected.md) — 09:00 boot order, not a bug · [Probe in pod with nc](reference_probe_staging_routes_from_inside_the_pod_with_nc.md) — no port-forward in sandbox; lookup-self body holds the token
- [Webhook 404 = page unbound](reference_staging_fb_webhook_404_is_page_unbound_silently.md) — no log line; read fb_app_audit_log
- [FB staging test app](reference_staging_fb_app_is_a_test_app.md) · [Sellsuki-New review](reference_sellsuki_new_meta_app_state.md) · [Model = provider perm](project_model_change_is_provider_level.md)
- [Docs on AI-49](reference_monorepo_docs_live_on_ai49_branch.md) · [control-tower diverged](reference_control_tower_data_js_diverged_main_vs_ai49.md)

## Sub-indexes — open when the topic comes up
- [Identity / CCS / rps](index_identity_ccs.md) — Kratos, Keto, rps, CCS config, local traps
- [BOLA](index_bola.md) — deploy, access model, LINE/LIFF, contact, APM
- [OC2Plus](index_oc2plus.md) — prod DB, member app, 3rd-party API, loyalty, consent
- [Personal / infra](index_infra.md) — machine, local stack, overmind, Caddy, rtk, DNS, runners
- [Git / CI](index_git_ci.md) — merge traps, submodules, CI, review bot · [auto-merge no CI](reference_glab_auto_merge_merges_immediately_without_ci_gate.md) · [Outdated deploy job](reference_gitlab_outdated_deployment_job_skips_the_newer_pipeline.md)
- [Patona / OMS / QMS / DS](index_patona_oms_ds.md) · [AI Chat Platform](index_ai_chat.md)
- [Resume dead subagent](reference_resume_dead_subagent_by_sendmessage_not_respawn.md) — SendMessage the same agentId
- [Team workflow goal](project_team_workflow_sharing_goal.md) · [Test users = infra](project_test_users_as_infrastructure.md) · [pstack](project_pstack_as_workflow_benchmark.md) · [SukiSpace vision](project_sukispace_marketplace_ais_vision.md)
- [dev onboarding dead ends](project_dev_onboarding_dead_ends_2026_10_11.md) — 2 unfixed blockers for new dev users
