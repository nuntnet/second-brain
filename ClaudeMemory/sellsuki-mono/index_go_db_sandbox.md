# Go / DB / sandbox / console-test traps (sub-index)

Moved out of MEMORY.md 2026-11-05 to keep the index under its read limit.

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
- [chat-core tests: one migrated DB](reference_chat_core_integration_tests_share_one_migrated_db.md) — migratedDB per test ~90-200s; subtests + fresh workspaces; assignee "" not NULL
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
- [Kit Member.Role = rps name](reference_kit_membership_role_is_rps_role_name.md) — company role is the rps NAME, direct grant in DirectWorkspaceRole; fixtures via MemberFromRoleTenants
- [Fiber Group = Use on prefix](reference_fiber_group_middleware_gates_every_later_route_under_prefix.md) — shared-prefix group gate leaks; 403 helper returns nil
- [worktree -b tracks origin/main](reference_worktree_add_b_tracks_origin_main_push_explicit_refspec.md) — push with explicit refspec
- [go test skips _dirs](reference_go_test_skips_underscore_dirs_ccs_migration_tests.md) — CCS _00NN migration tests by explicit path
- [Reverted merge = ancestor](reference_reverted_merge_branch_is_ancestor_reapply_by_reverting_the_revert.md) — merging a reverted card branch again is a no-op; revert the revert on a -v2 branch
- [Checkpoint snapshot = turn start](reference_chat_core_checkpoint_snapshot_is_turn_start_recompute_after_facts.md) — read after reply lags a turn; recompute with TriggerFactWrite

- [Pane can't boot mock console](reference_browser_pane_cannot_boot_console_mock_mode_msw.md) — MSW worker fails in pane; use Playwright on own port
- [Local chat-core = browser cookie](reference_local_chat_core_needs_kratos_browser_cookie.md) — Kratos browser flow → sellsuki_session; X-User-Id/token 401
- [zsh $VAR:refs breaks](reference_zsh_colon_modifier_eats_git_refspec.md) — write ${VAR}:refs/… and ${B}:path
- [Migration no. race at MR time](reference_migration_number_collides_with_main_at_wave_mr_time.md) — merge-tree vs origin/main first; CCS MIGRATE_ONLY = first prefix hit
- [Sandbox bind EPERM + slog panic](reference_sandbox_bind_eperm_fails_go_tests_and_slog_panics_in_tests.md) — listener tests FAIL in sandbox; slog.Warn panics in unit tests

- [Initial stage ≠ Cold](reference_lead_initial_stage_is_not_cold.md) — is_initial is "cold" only in the default template; mock uses "new"; hid every new lead (console !147)

- [Docker stop = AI MVP stack down](reference_docker_stop_takes_down_ai_mvp_stack_recovery.md) — Kafka exits 143, overmind ai-mvp gone; docker start kafka, nunt runs ai-mvp-restart.sh
- [Overmind start trap](project_overmind_restart_quirk.md) — `start <name> -D` on a running overmind leaves it dead; `restart -s <sock> <name>`; stack may be on .overmind-ai-mvp.sock

- [zsh `path` var = PATH](reference_zsh_variable_named_path_clobbers_PATH.md) — naming a variable `path` wipes PATH; "command not found" results are fake

- [fluid-ui skill: user-level, REVISE](project_sellsuki_fluid_ui_skill_user_level_eval_revise.md) — installed 11-04, eval REVISE; don't move into repo yet

- [gorm Raw scan + embedded model = empty](reference_gorm_raw_scan_embedded_model_fields_empty.md) — c.* into embedded model leaves fields blank; two reads · share one migrated schema in test/
- [Viewer = "operator" role](reference_me_workspaces_role_is_operator_for_viewer_gate_on_bundles.md) — /me/workspaces calls a viewer operator; gate console actions on AI-360 bundles

