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
- [Wave = tsc + full e2e](feedback_wave_integration_full_suite_and_tsc_on_merged_tree.md) — unit green hid a TS2308; cards' e2e green hid 14 fails; attribute by card-head worktree
- [Tests from card](feedback_tests_derive_from_the_card_not_the_code.md) — card = SSOT, never fit expected to code
- [Standing approval staging](feedback_standing_approval_merge_deploy_staging_after_gates.md) — gates green → merge/deploy staging myself · prod still ask
- [Wave/ledger first](feedback_check_wave_lists_and_ledger_before_starting_a_card.md) — grep waves/*.md + docs/cards/<CARD>.md before briefing; wave card = no MR
- [Bot before merge](feedback_read_review_bot_before_merging.md) — read the bot Result line; green+mergeable ≠ bot passed
- [Comment card](feedback_comment_jira_card_when_loop_closes.md) — Jira comment + status follows MR
- [MR after full ladder](feedback_open_mr_only_after_full_test_ladder_green.md) — unit + real-DB -p 1 + AC table proved red, then MR; kit…
- [Never cut AI reply](feedback_never_cut_ai_reply_gates_observe_only.md) — business-rule gates log, prompt enforces (D-060)
- [Urgent thread](feedback_keep_urgent_thread_visible.md) — don't bury live damage under a new topic
- [Sweep Codex branches](feedback_sweep_codex_branches_and_ship_whole.md) — Codex release branch is a superset; its BLOCKED ≠ PO blocker
- [Cap agents when waves parallel](feedback_parallel_wave_sessions_cap_agents_and_ladders.md) — ≤2 card agents, one ladder, e2e --workers=2; load hit ~150
- [Ask with full context](feedback_ask_decisions_with_full_context_not_bare_lists.md) — what/why/options+cost/recommendation
- [Console UX rules](feedback_console_ux_story_presets_icons_generic.md) — story/wizard for newcomers, presets not free text, icon+hover not labels, never one industry
- [Handoff ≠ state](feedback_handoff_claims_check_ledger_and_mr.md) — check ledger + open MRs before trusting "no code yet"
- [Check env yourself](feedback_check_read_only_env_facts_yourself_user_does_not_know.md) — port-forward rps + grpcurl, don't ask
- [Read Job logs first](feedback_read_job_logs_before_deleting_the_job.md) — save logs before deleting a Job
- [Morning retries in waves](feedback_morning_retries_in_waves_runner_cluster_is_12_nodes.md) — ≤6–8 jobs per wave, 12-node runner
- [Sprint sweep = deploy](feedback_sprint_sweep_board_column_means_deploy_not_ac_proof.md) — column means deployed, not AC proven
- [Mock review](feedback_mock_mode_ui_review_check_real_adapter_before_calling_defect.md) — check real adapter before calling a defect
- [rtk cat strips comments](feedback_rtk_cat_strips_comments_never_use_cat_to_rewrite_files.md) — write files with python bytes
- [rtk proxy to compare](feedback_compare_command_outputs_with_rtk_proxy.md) — rtk truncates output, hides merge commits in git log, masks go test exit codes; read raw
- [Vault/prod: hand over commands](user_has_vault_and_prod_access_agent_hands_over_commands.md) — nunt has Vault+prod access
- [Backticks in "..." execute](reference_backticks_in_double_quoted_python_c_execute.md) — markdown in python -c "…" ran a migration; text goes in a file
- [Inbox ids collide](reference_decision_inbox_ids_collide_across_branches.md) — inbox.py = max+1 of THIS branch; sweep all branches first

## Git / branches
- [Sparse docs worktree + reuse sibling merge](reference_sparse_docs_worktree_and_reuse_sibling_merge.md) — ledgers via --no-checkout + sparse docs/cards; reuse a pure wave+main merge
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
- [Go / DB / sandbox / test traps](index_go_db_sandbox.md) — sandbox, go cache, pgx/gorm, chat-core test DBs, console e2e ports, ssk-* components, merge traps

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
- [ssk-drawer named slots + closed h1](reference_ssk_drawer_named_slots_and_closed_header_h1.md) — body needs slot="body" (jsdom hides it); closed drawer = 2nd h1
- [Admin-fe e2e port collides](reference_admin_fe_e2e_fixed_port_parallel_sessions_collide.md) — 5196 hard-coded; temp config per session/verifier; -g is a regex
- [BOLA internal API header](reference_bola_internal_api_uses_x_system_token_not_x_service_token.md) — X-System-Token, not X-Service-Token
- [SskPageTitle children](reference_ssk_page_title_takes_children_not_title_prop.md) — `title=` renders an empty h1; tests stay green
- [Keep-both merge drops braces](reference_merge_conflict_keep_both_drops_shared_closing_lines.md) — rebuild base+ours+theirs; postcss parse; zsh `$REF:path`
- [Console e2e port 5196 shared](reference_console_e2e_port_5196_shared_use_temp_config.md) — temp config on 5197, app-local playwright
- [Prod port-forward denied](reference_prod_port_forward_denied_kubectl_get_logs_ok.md) — prod get/logs OK, port-forward refused; hand grpcurl to user
- [BOLA→QMS never exercised](project_bola_qms_quota_path_never_exercised.md) — enforcement off dev+staging, no bola quotas anywhere; runbook docs/release/qms
- [Console paging needs canvas scroll](reference_console_paging_footer_needs_canvas_scroll.md) — sentinel inside main.main-landmark; e2e scrolls the canvas; seeds need distinct times
- [Workflow doc + plugin regen](reference_workflow_doc_outline_and_plugin_regen.md) — Outline id, no H1 in update text, build-plugin.py --check not hand diff
- [Scheduled run can't merge](reference_scheduled_task_cannot_merge_classifier_refuses.md) — classifier refuses merge unattended; merge in a live turn
- [Develop merge deletes branch](reference_develop_merge_deletes_source_branch_repush_for_main.md) — main MR 'source_branch does not exist' → re-push merged SHA
- [CCS2 cache + dev 401](reference_ccs2_staging_index_html_cached_dev_mgmt_401.md) — staging index.html cached (use ?v=), dev mgmt API 401, test QMS pages on staging
- [Permission matrix row](reference_chat_core_permission_matrix_needs_every_new_route.md) — every new workspace route needs a route(...) line in permission_matrix_test.go
- [Ladder dies with the turn](reference_background_ladder_dies_with_the_turn_use_nohup.md) — long ladders as nohup script + done file
- [Ladder survives drop](reference_ladder_survives_network_drop_check_before_restart.md) — check done-file/pgrep before restarting; two runs on one test DB = bogus FAILs
- [BOLA test/api red on main](reference_bola_test_api_broadcast_mock_panic_on_main.md) — CountBroadcasts mock panic is baseline (b2d0f47)
- [chat-core coverage = in-package only](reference_chat_core_storage_coverage_counts_in_package_tests_only.md) — storage_coverage has no -coverpkg; ./test/ adds 0; external _test pkg in same dir counts
