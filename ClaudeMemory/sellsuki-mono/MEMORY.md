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
- [Prove-red no head](feedback_prove_red_script_never_pipe_into_head.md) — head kills the restore; commit first; zsh needs ${=pkgs}
- [Tests from card](feedback_tests_derive_from_the_card_not_the_code.md) — card = SSOT, never fit expected to code
- [Standing approval staging](feedback_standing_approval_merge_deploy_staging_after_gates.md) — gates green → merge/deploy staging myself · prod still ask
- [Wave/ledger first](feedback_check_wave_lists_and_ledger_before_starting_a_card.md) — grep waves/*.md + docs/cards/<CARD>.md before briefing; wave card = no MR
- [Bot before merge](feedback_read_review_bot_before_merging.md) — read the bot Result line; green+mergeable ≠ bot passed
- [Comment card](feedback_comment_jira_card_when_loop_closes.md) — Jira comment + status follows MR
- [MR after full ladder](feedback_open_mr_only_after_full_test_ladder_green.md) — unit + real-DB -p 1 + AC table proved red, then MR; kit…
- [Never cut AI reply](feedback_never_cut_ai_reply_gates_observe_only.md) — business-rule gates log, prompt enforces (D-060)
- [Urgent thread](feedback_keep_urgent_thread_visible.md) — don't bury live damage under a new topic
- [Sweep Codex branches](feedback_sweep_codex_branches_and_ship_whole.md) — Codex release branch is a superset; its BLOCKED ≠ PO blocker
- [Ask with full context](feedback_ask_decisions_with_full_context_not_bare_lists.md) — what/why/options+cost/recommendation
- [Check env yourself](feedback_check_read_only_env_facts_yourself_user_does_not_know.md) — port-forward rps + grpcurl, don't ask
- [Read Job logs first](feedback_read_job_logs_before_deleting_the_job.md) — save logs before deleting a Job
- [Morning retries in waves](feedback_morning_retries_in_waves_runner_cluster_is_12_nodes.md) — ≤6–8 jobs per wave, 12-node runner
- [Sprint sweep = deploy](feedback_sprint_sweep_board_column_means_deploy_not_ac_proof.md) — column means deployed, not AC proven
- [Mock review](feedback_mock_mode_ui_review_check_real_adapter_before_calling_defect.md) — check real adapter before calling a defect
- [rtk cat strips comments](feedback_rtk_cat_strips_comments_never_use_cat_to_rewrite_files.md) — write files with python bytes
- [rtk proxy to compare](feedback_compare_command_outputs_with_rtk_proxy.md) — rtk truncates output, hides merge commits in git log; read raw
- [Vault/prod: hand over commands](user_has_vault_and_prod_access_agent_hands_over_commands.md) — nunt has Vault+prod access
- [Backticks in "..." execute](reference_backticks_in_double_quoted_python_c_execute.md) — markdown in python -c "…" ran a migration; text goes in a file
- [Inbox ids collide](reference_decision_inbox_ids_collide_across_branches.md) — inbox.py = max+1 of THIS branch; sweep all branches first

## Git / branches
- [Stacked promotion wave](reference_stacked_promotion_wave_develop_to_main.md) — chain-merge in scratch worktree, user runs merge script
- [Auto mode blocks MR create](reference_auto_mode_blocks_glab_mr_create.md) — prepare JSON + create-mrs.sh · project ids inside
- [OC2Plus develop↔main diverged](reference_oc2plus_backoffice_develop_main_diverged_2026_10_11.md) — OC-4736 one-time reunion
- [rps/mgmt-backend lines](reference_rps_and_management_backend_develop_main_diverged.md) — rps not diverged, mgmt-backend is
- [Messaging develop↔main diverged](reference_messaging_backend_develop_main_diverged_oc4583.md) — main-cut MR into develop conflicts (OC-4583) · never…
- [Worktree from main lacks AI repos](reference_monorepo_worktree_from_main_lacks_ai_submodules_and_tooling.md) — detach-worktree each submodule; land ledgers on AI-49 with…
- [Night = staging-th only](reference_night_window_is_staging_th_only_production_runners_run.md) — prod runners still run
- [First CI run of old specs](reference_first_ci_run_of_never_run_playwright_specs_finds_real_bugs_and_axe_baseline_drift.md) — real bugs + axe drift · never rebase pushed

## Projects / state
- [PO pack not on main](project_po_workflow_pack_not_on_main_2026_11_02.md) — /po-team only on AI-49 · PO pack = monorepo MR !52 (open, not merged) · engineering half has no MR
- [eval-skill pitfalls](reference_eval_skill_design_pitfalls_2026_10_12.md) — real tier-2 task, strip evidence from baselines, 429 kills agents
- [nunt-mode profile](project_nunt_mode_agent_profile.md) — /my-mode, eval REVISE · round 3 (tier 2) incomplete, rerun after spend limit
- [Tiers + Decision Inbox](project_workflow_tiers_and_inbox_2026_10_13.md) — AC tiers A/B/C + question queue
- [Inbox ids collide](reference_inbox_py_ids_collide_across_branches.md) — inbox.py is max+1 per branch; pin the coordinator's id via load/save
- [OC-4739 file-service backfill](project_oc4739_file_service_access_backfill.md) — staging 14,749 companies · AC8 left
- [Oc2plus SaaS OC-4763](project_oc2plus_saas_plan_epic_oc4763_decisions_2026_10_16.md) — PO quota/billing decisions 10-16
- [System testing direction](project_system_testing_direction_2026_10_11.md) — invariants suite, seed-scenario, ratchet
- [Mahanakhon decisions](project_mahanakhon_proposal_vs_roadmap_decisions_2026_10_14.md) — CRM owns active/tier, per company
- [Playbook/Goal/Lead/KB cards](project_playbook_goal_lead_kb_cards_2026_10_24.md) — AI-312..327 + PO verdicts
- [Wave 1 state 10-25](project_wave1_playbook_goal_lead_kb_state_2026_10_25.md) — 8 cards pushed, no MR/verifier
- [AI-318 depends on model](project_ai318_interest_gate_depends_on_the_model_2026_10_25.md) — gemini fails, gpt-4o-mini passes · D-038
- [AI chat dev order 10-27](reference_ai_chat_dev_order_reconcile_2026_10_27.md) — 5 waves · both sprint cleanups applied…
- [AI chat permission staging 10-27](project_ai_chat_staging_permission_state_2026_10_27.md) — Owner 3 generations · no usage/billing preset · switcher…
- [Settings redesign v9](project_ai_chat_settings_redesign_v9_approved_2026_10_31.md) — v9 approved 10-31 · AI-357..360 cut · D-095 open · resume…
- [chat-core not on prod](project_chat_core_not_deployed_to_production.md) — chat-core cards have no real prod step yet
- [Unscheduled sweeps AI-351](project_chat_core_unscheduled_sweeps_ai351.md) — 9 chat-core sweeps had no CronJob · rolling-summary added…
- [Messaging chat flip 10-27](reference_messaging_chat_module_staging_flip_2026_10_27.md) — off by !80 09:59, back on by !82 16:46 (deploy 275941)…

## Jira
- [Sweep via files](reference_jira_board_sweep_via_saved_results.md) · [BOLA](reference_bola_jira_project.md) · [OC2Plus](reference_oc2plus_jira_project.md) · [Patona](reference_pat_jira_project.md) · [Sprint ids](reference_jira_sprint_ids_not_contiguous.md) · [PAT sprints](reference_pat_board_sprints.md) · [PAT epics](project_pat_epic_links_unwired.md) · [Crosses sessions](reference_jira_mcp_crosses_responses_between_sessions.md) · [Quirks](reference_jira_mcp_search_quirks.md) · [ADF Thai](reference_jira_editissue_adf_breakage.md) · [No fallback](reference_no_local_jira_fallback.md) · [403 writes](reference_jira_mcp_writes_403_midsession.md) · [BOLA no In Review](reference_bola_jira_has_no_in_review_status_and_auto_mode_denies_merge.md)

## Go / DB / sandbox
- [ns octoplus](reference_oc2plus_cluster_namespace_is_octoplus.md) · [Files≠schema](reference_migration_files_are_not_applied_schema.md) · [envDefault](reference_envdefault_localhost_masks_missing_config.md) · [goqu](reference_goqu_dialect_blank_import.md) · [AutoMigrate](reference_gorm_pgx_libpq_automigrate.md) · [Drops false](reference_gorm_updates_drops_false.md) · [Lease](reference_lease_claim_ownership_bug_class.md) · [Partial idx](reference_pg_partial_index_onconflict_generic_plan.md) · [404 open](reference_ambiguous_404_fail_open.md) · [Own guard](reference_auth_endpoint_behind_own_guard.md) · [Kafka silent](reference_kafka_silent_publish_failure.md) · [Fiber ctx](reference_fiber_ctx_strings_corrupt_prometheus_labels.md)
- [pgx cached plan](reference_pgx_cached_plan_breaks_old_pods_after_add_column_migration.md) — old pods error after ADD COLUMN
- [Sandbox: .claude/commands+skills, worktree add](reference_sandbox_blocks_submodule_claude_commands_skills_and_worktree_add.md) — write-protected; worktree add needs unsandboxed
- [ai-agent company + charge_class](reference_ai_agent_needs_registered_company_charge_class_from_tier_path.md) — no empty company; mark spend via tier_path
- [Session worktree Write hook](reference_session_worktree_hook_blocks_writes_outside_it.md) — submodule worktrees go INSIDE the session worktree
- [pre-push false FAILED](reference_pre_push_hook_false_ship_check_failed.md) — hook crashes (args[@] unbound) and prints ship-check…
- [Sandbox: DB tests skip](reference_sandbox_blocks_localhost_so_db_tests_skip_silently.md) · [Sandbox: Go CLI TLS](reference_sandbox_blocks_go_cli_tls_and_ssh.md) · [TMPDIR differs](reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash.md) — use scratchpad absolute path
- [fd exhaustion = codegraph](reference_codegraph_watchers_exhaust_system_fds.md) — "too many open files in system" machine-wide; user kills…
- [Local Mongo no replset → consent 503](reference_local_mongo_standalone_blocks_consent_acceptance_otp_members.md) — 4 causes: replset, write flags, 7 indexes, consentee…
- [Go toolchain cache damaged after crash](reference_go_toolchain_cache_damaged_after_fd_crash_use_gotoolchain_local.md) — "os/exec is not in std" → GOTOOLCHAIN=local; user deletes…
- [Parallel sessions share test Postgres](reference_parallel_sessions_share_test_postgres_spurious_route_test_failures.md) — use your own DB via CHAT_CORE_TEST_POSTGRESQL_URI (go run to create); drop after
- [go-build cache corrupt](reference_corrupt_go_build_cache_package_not_in_std.md) — "io/fs is not in std" → GOCACHE=scratchpad, don't wipe…
- [Kit pseudo-version pin](reference_chat_core_pins_unmerged_kit_by_pseudo_version.md) — go get kit@sha so chat-core CI builds before the tag
- [Disk full → Docker dies](reference_disk_full_kills_docker_desktop_go_cache_is_the_first_thing_to_delete.md) — delete go cache first
- [codegraph ENFILE](reference_codegraph_mcp_watchers_exhaust_macos_file_table.md) — watchers eat the system file table; lsof -c node, ask…
- [go cache after ENFILE](reference_go_build_cache_corrupt_after_enfile_io_fs_not_in_std.md) — "io/fs is not in std" = corrupt shared cache; private…

- [ENFILE corrupts go cache](reference_enfile_outage_corrupts_go_build_cache_and_kills_air_watchers.md) — air rebuilds fail silently, old binaries served; go clean -cache + overmind restart -s
- [Lead-vocab guard trap](reference_chat_core_lead_vocabulary_guard_substring_trap.md) — chat-core guard bans warm/hot/cold and substring "stage" (TestAgent!)
- [ai-agent confidence = 3 bands](reference_ai_agent_confidence_score_is_three_band.md) — Score is 0.1/0.35/0.95; thresholds in (0.35,0.95] all behave alike (D-193)
- [Pre-push blocks fake creds](reference_gstack_prepush_blocks_fake_credentials_in_tests.md) — assemble test secrets at run time, no --no-verify

## Sub-indexes — open when the topic comes up
- [AI chat / KB](index_ai_chat.md) — KB eval, staging recipes, console, rag-core, Vault (incl. everything formerly listed here)
- [Tests](index_tests.md) — slog, testify, Robot, axe, jsdom, DB-test -p 1, scenario harness, prompt A/B
- [Identity / CCS / rps](index_identity_ccs.md) — Kratos, Keto, rps, CCS config, local traps
- [BOLA](index_bola.md) — deploy, access model, LINE/LIFF, contact, APM
- [OC2Plus](index_oc2plus.md) — prod DB, member app, 3rd-party API, loyalty, consent
- [Personal / infra](index_infra.md) — machine, local stack, overmind, Caddy, rtk, DNS, runners
- [Git / CI](index_git_ci.md) — merge traps, submodules, CI, review bot · [auto-merge no CI](reference_glab_auto_merge_merges_immediately_without_ci_gate.md) · [Outdated deploy job](reference_gitlab_outdated_deployment_job_skips_the_newer_pipeline.md)
- [Patona / OMS / QMS / DS](index_patona_oms_ds.md) · [AI Chat Platform](index_ai_chat.md)
- [Resume dead subagent](reference_resume_dead_subagent_by_sendmessage_not_respawn.md) — SendMessage the same agentId
- [Team workflow goal](project_team_workflow_sharing_goal.md) · [Test users = infra](project_test_users_as_infrastructure.md) · [pstack](project_pstack_as_workflow_benchmark.md) · [SukiSpace vision](project_sukispace_marketplace_ais_vision.md)
- [dev onboarding dead ends](project_dev_onboarding_dead_ends_2026_10_11.md) — 2 unfixed blockers for new dev users
- [Verifier exec denied](reference_verifier_session_exec_and_port_forward_denied_even_with_relayed_approval.md) — exec + port-forward denied even with relayed approval…
- [Usage spool stuck](project_staging_ai_agent_usage_spool_stuck_poison_event_2026_10_29.md) — staging ai-agent replay fails, QMS no plan for company…
- [RAG probes: user mints token](reference_rag_staging_probes_user_mints_bff_token_port_forward_ok.md) — exec denied, port-forward OK, never read BFF secret…
- [Pane file upload](reference_browser_pane_file_upload_via_datatransfer.md) — DataTransfer on input[type=file]; conversation URL is…
- [Parallel Bash cd / zsh split](reference_parallel_bash_cd_moves_git_checkout_into_shared_root_use_git_dash_C.md) — use `git -C <abs>` + `bash -c` loops; a parallel `cd` once…
- [Company-mgmt FE tests](reference_company_mgmt_frontend_vitest_recipe_and_shared_node_modules.md) — vitest 1.6 pool flags, rtk breaks npx vitest, symlink…
- [Sandbox go cache + BSD sed](reference_sandbox_blocks_go_build_cache_and_bsd_sed_noop_mutation.md) — go vet needs unsandboxed; sed -i no-op fakes a failed…
- [Admin-fe worktree install + key-count tripwire](reference_admin_fe_worktree_install_and_i18n_key_count_tripwire.md) — own pnpm install; LabelPack test counts i18n keys
- [Worktree tracks main → push goes nowhere](reference_worktree_add_from_origin_main_tracks_main_push_goes_nowhere.md) — `--no-track`, push HEAD:refs/heads/<b>, check ls-remote
- [React 19 boolean ARIA on ssk-* = ""](reference_react19_boolean_aria_on_custom_elements_renders_empty.md) — use ariaBool() strings
- [ssk-drawer closes on Escape itself](reference_ds_ssk_drawer_closes_on_escape_itself.md) — no app Escape listener; tooltip has only private _isOpen
- [BOLA internal API header](reference_bola_internal_api_uses_x_system_token_not_x_service_token.md) — X-System-Token, not X-Service-Token
