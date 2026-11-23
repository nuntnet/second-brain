# Memory Index

Titles are 1–3 words on purpose — each file's `description:` carries the real detail. Always open the file before relying on it. Domain detail lives in the sub-indexes at the bottom; open the one for the topic at hand.

## Feedback & how to work
- [Prior art: all Jira projects](feedback_prior_art_sweep_every_jira_project_not_just_target.md) — search PAT+AI+OC, read adjacent cards' content not titles
- [RBAC no-backfill plan](project_rbac_preset_backfill_cards_pat2872_2873.md) — all on PAT epic PAT-2874 (moved from AI board 11-17); CCS reconciler on develop only, rps on main
- [Internal hosts need VPN](reference_outline_mcp_vpn_blocker.md) — docs.sellsuki.com + outline MCP = WireGuard VPN only; `CONNECT_TIMEOUT` at start = VPN off → ask user to connect + Reconnect outline · MST collection hidden from list_collections · [VPN clients](reference_machine_vpn_clients_wireguard_is_company_vpn.md)
- [Thai](user_language_thai.md) · [Act](feedback_decisive_deep_execution.md) · [Self-explain](feedback_selfexplaining_ux.md) · [Subagents](feedback_subagent_model_tiering.md) · [PMM voice](feedback_product_marketing_voice.md) · [Design fidelity](feedback_design_handoff_full_fidelity.md) · [Entity pure](feedback_entity_pure_zod_dto.md)
- [Card=story](feedback_card_user_story_flows.md) · [No mid-sprint](feedback_no_scope_change_in_sprint.md) · [QA guide](feedback_qa_reproduce_guide.md) · [Cite line](feedback_ground_claims_file_line.md) · [Report not edit](feedback_report_wrong_cards_dont_edit.md) · [Fix reaches all](feedback_fix_must_reach_everyone.md) · [Central≠caller](feedback_central_service_no_caller_domain.md)
- [Reproduce](feedback_reproduce_the_number_before_asking.md) · [As user](feedback_verify_as_the_user_sees_it.md) · [Norm first](feedback_check_the_norm_before_calling_it_broken.md) · [Grep service](feedback_grep_the_spec_is_not_grep_the_service.md) · [Before GAP](feedback_search_before_declaring_gap.md) · [Absence](feedback_verify_absence_claims.md) · [head=sample](feedback_head_on_grep_is_sampling_not_verification.md) · [All layers](feedback_search_the_whole_stack_not_one_layer.md)
- [No bug card](feedback_no_bug_card_for_in_sprint_fixes.md) · [Codex](feedback_user_runs_codex_in_parallel.md) · [Push often](feedback_commit_push_along_the_way.md) · [Git flow](feedback_git_flow_main_develop_main.md) (main→develop test→same branch to main) · [No promo MR](feedback_no_develop_to_main_promotion_mrs.md) · [Git safety](feedback_parallel_sessions_git_safety.md) · [List MRs](feedback_list_open_mrs_before_opening_one.md) · [OC↔PAT](feedback_oc_pat_board_ownership_rule.md) · [OC→develop](feedback_oc2plus_merge_to_develop.md) (partly superseded) · [PIS FF](feedback_ff_only_force_push_ok.md) · [No tiny cards](feedback_no_tiny_cards_bundle_as_ac.md)
- [Generator no rmtree](feedback_generator_must_not_wipe_shared_dirs.md) — write only marker files
- [Nested po-lead](feedback_nested_po_lead_loses_drafts_write_to_files.md) — spawn writers from main session, write to files
- [gemini-2.5-flash only](feedback_gemini_25_flash_is_the_kb_answer_model.md) — KB answer model fixed 10-07
- [Prove-red no head](feedback_prove_red_script_never_pipe_into_head.md) — head kills the restore; commit first; zsh needs ${=pkgs}
- [Wave = tsc + full e2e](feedback_wave_integration_full_suite_and_tsc_on_merged_tree.md) — unit green hid a TS2308; cards' e2e green hid 14 fails; attribute by card-head worktree
- [Journey first](feedback_journey_first_before_screens_on_multi_card_features.md) — multi-card feature: journey + probe table + prototype before screens; no screenshot-by-screenshot fixes
- [Probe every control](feedback_probe_every_control_against_real_service_before_ui.md) — control × entity table from the real service before UI; one 200 ≠ verified
- [Tests from card](feedback_tests_derive_from_the_card_not_the_code.md) — card = SSOT, never fit expected to code
- [Standing approval staging](feedback_standing_approval_merge_deploy_staging_after_gates.md) — gates green → merge/deploy staging myself · prod still ask
- [MR per wave, not per card](feedback_open_mr_per_wave_or_batch_not_per_card.md) — nunt 11-10 on the PAT5 BOLA set; scope beyond it not confirmed
- [Wave/ledger first](feedback_check_wave_lists_and_ledger_before_starting_a_card.md) — grep waves/*.md + docs/cards/<CARD>.md before briefing; wave card = no MR
- [Bot before merge](feedback_read_review_bot_before_merging.md) — read the bot Result line; green+mergeable ≠ bot passed
- [Comment card](feedback_comment_jira_card_when_loop_closes.md) — Jira comment + status follows MR
- [MR after full ladder](feedback_open_mr_only_after_full_test_ladder_green.md) — unit + real-DB -p 1 + AC table proved red, then MR; kit…
- [Never cut AI reply](feedback_never_cut_ai_reply_gates_observe_only.md) — business-rule gates log, prompt enforces (D-060)
- [Urgent thread](feedback_keep_urgent_thread_visible.md) — don't bury live damage under a new topic
- [Sweep Codex branches](feedback_sweep_codex_branches_and_ship_whole.md) — Codex release branch is a superset; its BLOCKED ≠ PO blocker
- [Cap agents when waves parallel](feedback_parallel_wave_sessions_cap_agents_and_ladders.md) — ≤2 card agents, one ladder, e2e --workers=2; load hit ~150
- [Ask with full context](feedback_ask_decisions_with_full_context_not_bare_lists.md) — what/why/options+cost/recommendation
- [Console v9 = mess, reset first](feedback_console_v9_frontend_is_a_mess_reset_before_more_screens.md) — no more screens until audit + IA + visual direction agreed with nunt
- [Console UX rules](feedback_console_ux_story_presets_icons_generic.md) — story/wizard for newcomers, presets not free text, icon+hover not labels, never one industry
- [Data-page prototype review](feedback_prototype_review_controls_feasibility_date_ranges.md) — every control live (scripted), feasibility per number on the page, per-customer cycles/date range, one summary + one tab row
- [Other agents' runbooks](feedback_review_other_agents_runbooks_against_git_rules.md) — Grokbot runbook said rebase+force-push and stale "known red"; GET GitLab + translate to merge-main first
- [Handoff ≠ state](feedback_handoff_claims_check_ledger_and_mr.md) — check ledger + open MRs before trusting "no code yet"
- [Check env yourself](feedback_check_read_only_env_facts_yourself_user_does_not_know.md) — port-forward rps + grpcurl, don't ask
- [Agents never reset --hard](feedback_agents_never_reset_hard_cd_guard.md) — a subagent reset the monorepo ROOT 9× (failed cd) 11-19; briefs forbid destructive git, `cd X || exit`, merge-tree for trials
- [Kill by PID, never pkill](feedback_never_pkill_pattern_kill_by_pid_on_macos.md) — BSD pkill reads a trailing -n as a pattern; a verifier killed Docker Desktop + apps
- [Read Job logs first](feedback_read_job_logs_before_deleting_the_job.md) — save logs before deleting a Job
- [Morning retries in waves](feedback_morning_retries_in_waves_runner_cluster_is_12_nodes.md) — ≤6–8 jobs per wave, 12-node runner
- [Sprint sweep = deploy](feedback_sprint_sweep_board_column_means_deploy_not_ac_proof.md) — column means deployed, not AC proven
- [Mock review](feedback_mock_mode_ui_review_check_real_adapter_before_calling_defect.md) — check real adapter before calling a defect
- [rtk cat strips comments](feedback_rtk_cat_strips_comments_never_use_cat_to_rewrite_files.md) — write files with python bytes
- [rtk proxy to compare](feedback_compare_command_outputs_with_rtk_proxy.md) — rtk truncates output, hides merge commits in git log, masks go test exit codes; read raw
- [Vault CLI default = local docker](reference_vault_cli_default_is_local_docker_vault.md) — unset VAULT_ADDR writes to 127.0.0.1:8200 (docker); staging = VPN URL or Teleport port-forward + OIDC; force-sync ExternalSecret
- [Vault/prod: hand over commands](user_has_vault_and_prod_access_agent_hands_over_commands.md) — nunt has Vault+prod access
- [Backticks in "..." execute](reference_backticks_in_double_quoted_python_c_execute.md) — markdown in python -c "…" ran a migration; text goes in a file
- [Inbox ids collide](reference_decision_inbox_ids_collide_across_branches.md) — inbox.py = max+1 of THIS branch; sweep all branches first

## Git / branches
- [Submodule worktree inside session wt](reference_session_worktree_hook_put_submodule_worktree_inside.md) — Write hook blocks other worktrees; nest it under the session worktree
- [Sparse docs worktree + reuse sibling merge](reference_sparse_docs_worktree_and_reuse_sibling_merge.md) — ledgers via --no-checkout + sparse docs/cards; reuse a pure wave+main merge
- [Stacked promotion wave](reference_stacked_promotion_wave_develop_to_main.md) — chain-merge in scratch worktree, user runs merge script
- [Commit only my lines of a shared dirty file](reference_commit_only_my_lines_of_a_shared_dirty_file_via_update_index.md) — INBOX.md carries other sessions' rows; hash-object + update-index --cacheinfo, commit with no pathspec
- [Auto mode blocks MR create](reference_auto_mode_blocks_glab_mr_create.md) — prepare JSON + create-mrs.sh · project ids inside
- [OC2Plus develop↔main diverged](reference_oc2plus_backoffice_develop_main_diverged_2026_10_11.md) — OC-4736 one-time reunion
- [rps/mgmt-backend lines](reference_rps_and_management_backend_develop_main_diverged.md) — rps not diverged, mgmt-backend is
- [Messaging develop↔main diverged](reference_messaging_backend_develop_main_diverged_oc4583.md) — main-cut MR into develop conflicts (OC-4583) · never…
- [Worktree from main lacks AI repos](reference_monorepo_worktree_from_main_lacks_ai_submodules_and_tooling.md) — detach-worktree each submodule; land ledgers on AI-49 with…
- [Night = staging-th only](reference_night_window_is_staging_th_only_production_runners_run.md) — prod runners still run
- [First CI run of old specs](reference_first_ci_run_of_never_run_playwright_specs_finds_real_bugs_and_axe_baseline_drift.md) — real bugs + axe drift · never rebase pushed

## Projects / state
- [BOLA email + wave 11-19](project_bola_invite_email_via_messaging_and_wave_2026_11_19.md) — email via messaging-backend as sellsuki.system:bola-service (BOLA-424); bugs 405-423 + 404/424/311 = one wave, one release
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

- [SukiSpace subscribe branch-only](project_sukispace_subscribe_only_on_integration_branch_2026_11_20.md) — SG "Done" but code only on feature/sukispace-integration; no BOLA/AI plans
- [AI cost page no margin](project_provider_ai_cost_page_usd_only_no_margin_2026_11_20.md) — CCS2 /ai/usage USD only; revenue/FX missing (AI-461, AI-202)

- [Workspace = store+BOLA+chat](project_workspace_is_multi_product_ccs2_support.md) — CCS2 needs CCS3's unified workspace view for provider-admin support; OC-4263 BOLA-only is outdated
- [Dashboard+usage need pipeline prod](project_ai_chat_dashboard_and_usage_need_data_pipeline_prod.md) — dashboard-api not on prod + deny-all; URL alone = 403 (AI-498)

## Jira
- [Sprint ids + agile ops](reference_jira_sprint_ids_not_contiguous.md) — ids global across boards; f9f106ba connector `manageJiraSprint`/`listJiraBoardSprints` replaces the browser; AI S25–27 = 3307–3309
- [Sweep via files](reference_jira_board_sweep_via_saved_results.md) · [BOLA](reference_bola_jira_project.md) · [OC2Plus](reference_oc2plus_jira_project.md) · [Patona](reference_pat_jira_project.md) · [Sprint ids](reference_jira_sprint_ids_not_contiguous.md) · [PAT sprints](reference_pat_board_sprints.md) · [PAT epics](project_pat_epic_links_unwired.md) · [Crosses sessions](reference_jira_mcp_crosses_responses_between_sessions.md) · [Quirks](reference_jira_mcp_search_quirks.md) · [ADF Thai](reference_jira_editissue_adf_breakage.md) · [No fallback](reference_no_local_jira_fallback.md) · [403 writes](reference_jira_mcp_writes_403_midsession.md) · [BOLA no In Review](reference_bola_jira_has_no_in_review_status_and_auto_mode_denies_merge.md)

- [Thai Jira edits drift](reference_jira_bulk_thai_edits_drift_use_ascii_escaped_json.md) — re-typed text changes words; paste ASCII-escaped JSON, diff Jira vs file after
- [editJiraIssue fields rejected](reference_jira_mcp_editissue_fields_rejected_as_string.md) — schema-less tool rejects `fields`; the d4aa0ce4 connector's editJiraIssue works for descriptions
- [Meta app prod state 11-14](project_meta_app_sellsuki_a_prod_state_2026_11_14.md) — feed now Subscribed (seen 11-23, Page object), webhook v27 vs Graph v25, login config 3 perms, all Standard access

- [Docker reset → rebuild infra](reference_docker_reset_rebuild_local_infra.md) — images+named volumes gone, data/ survives; postgis apt archive, minio withdrawn→chainguard, rag-core volume, file-service ff
- [Duplicate local chat-core + Procfile reload](reference_local_duplicate_overmind_chat_core_and_procfile_reload.md) — stray .overmind-chat-core.sock eats the queue; restart never rereads Procfile → ai-mvp-restart.sh
- [overmind start -D ≠ one process](reference_overmind_start_d_does_not_start_one_process_use_restart.md) — after stop, only `overmind restart <name> -s <sock>` brings it back; rule file is wrong here

- [Console CSS: lint vs ratchet](reference_console_new_css_lint_vs_reset_ratchet.md) — new CSS can use neither --ssk-spacing-N nor --spacing-*; Tailwind layout; no info alert, strings ≤120
- [KB review POW via rehearsal](reference_console_kb_review_unavailable_in_mock_use_rehearsal.md) — review port UNAVAILABLE in mock; ?kb-range-review-rehearsal on e2e dev server

- [Vite stale proxy after checkout](reference_vite_stale_proxy_after_checkout_switch.md) — CCS2 "Something went wrong" = Vite kept old /ccs-api→:8092 proxy, no X-User-Id → 401; overmind restart web-provider

## Go / DB / sandbox
- [Unsandboxed TMPDIR shared](reference_unsandboxed_tmpdir_shared_log_names_collide.md) — sandbox-off $TMPDIR is /var/folders shared by all sessions; unit.log collided 11-20; log to scratchpad with a unique name
- [Go rag-core needs go1.25](reference_go_rag_core_needs_go125_toolchain_sonic.md) — sonic v1.14.1 breaks on go1.26 as [build failed]; GOTOOLCHAIN=go1.25.0
- [chat-core suite under load](reference_chat_core_local_suite_under_load_and_testdb_leftover.md) — timeouts at load ~30 → re-run failed pkgs -p 1; drop the leftover <db>_tp_<hash>
- [rag-core branch fixtures](reference_capture_rag_core_branch_fixtures_via_git_archive.md) — git archive + rag-core .venv, drive real ASGI routes; Write hook = stay inside session worktree
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
- [Env / sandbox / test / worktree traps](index_traps_tail.md) — verifier exec denied, sandbox glab/go/TMPDIR, ssk-* DS quirks, e2e ports, ladders, worktree/merge traps, console login
- [Providers patona/poshmedica](reference_providers_patona_poshmedica_company_pickers.md) — prod has no "sellsuki" provider yet (PAT-1529 renames patona); each app's company picker filters its own provider
- [Prod onboarding state 11-17](project_ai_chat_prod_onboarding_state_2026_11_17.md) — CCS prod v1.7.0, Owner chat perms granted by hand, re-grant CronJob suspended, AI-466 replaces OC2Plus route, trial-only plan defaults
- [Local console logout](reference_local_console_logout_needs_kratos_ui_main.md) — local kratos-ui-go old branch breaks it; use accounts.sellsuki.local/logout; replay with a session before claiming fixed
- [Console prod unit job flaky](reference_console_prod_unit_job_flaky_on_prod_runner.md) — v1.0.4 needed 4 runs on the prod runner; two flaky tests named
