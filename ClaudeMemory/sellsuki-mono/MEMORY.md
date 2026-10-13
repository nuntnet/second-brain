# Memory Index

Titles are 1–3 words on purpose — each file's `description:` carries the real detail. Always open the file before relying on it. Domain detail lives in the sub-indexes at the bottom; open the one for the topic at hand.

## Feedback & how to work
- [Thai](user_language_thai.md) · [Act](feedback_decisive_deep_execution.md) · [Self-explain](feedback_selfexplaining_ux.md) · [Subagents](feedback_subagent_model_tiering.md) · [PMM voice](feedback_product_marketing_voice.md) · [Design fidelity](feedback_design_handoff_full_fidelity.md) · [Entity pure](feedback_entity_pure_zod_dto.md)
- [Card=story](feedback_card_user_story_flows.md) · [No mid-sprint](feedback_no_scope_change_in_sprint.md) · [QA guide](feedback_qa_reproduce_guide.md) · [Cite line](feedback_ground_claims_file_line.md) · [Report not edit](feedback_report_wrong_cards_dont_edit.md) · [Fix reaches all](feedback_fix_must_reach_everyone.md) · [Central≠caller](feedback_central_service_no_caller_domain.md)
- [Reproduce](feedback_reproduce_the_number_before_asking.md) · [As user](feedback_verify_as_the_user_sees_it.md) · [Norm first](feedback_check_the_norm_before_calling_it_broken.md) · [Grep service](feedback_grep_the_spec_is_not_grep_the_service.md) · [Before GAP](feedback_search_before_declaring_gap.md) · [Absence](feedback_verify_absence_claims.md) · [head=sample](feedback_head_on_grep_is_sampling_not_verification.md) · [All layers](feedback_search_the_whole_stack_not_one_layer.md)
- [No bug card](feedback_no_bug_card_for_in_sprint_fixes.md) · [Codex](feedback_user_runs_codex_in_parallel.md) · [Push often](feedback_commit_push_along_the_way.md) · [Git flow](feedback_git_flow_main_develop_main.md) — แตกจาก main → develop เทส → branch เดิมเข้า main · [No promo MR](feedback_no_develop_to_main_promotion_mrs.md) · [Git safety](feedback_parallel_sessions_git_safety.md) · [List MRs](feedback_list_open_mrs_before_opening_one.md) · [OC↔PAT](feedback_oc_pat_board_ownership_rule.md) · [OC→develop](feedback_oc2plus_merge_to_develop.md) (แทนที่บางส่วน) · [PIS FF](feedback_ff_only_force_push_ok.md) · [No tiny cards](feedback_no_tiny_cards_bundle_as_ac.md)
- [generator ห้าม rmtree](feedback_generator_must_not_wipe_shared_dirs.md) — 10-11 build-plugin ลบ plugin ของ session อื่น · เขียนเฉพาะไฟล์ที่มี marker
- [po-lead ซ้อนชั้นทำร่างหาย](feedback_nested_po_lead_loses_drafts_write_to_files.md) — spawn writer จาก main session ให้เขียนลงไฟล์ก่อน publish
- [gemini-2.5-flash เท่านั้น](feedback_gemini_25_flash_is_the_kb_answer_model.md) — 2026-10-07 ปิด phase เลือก model · optimize ด้วย gemini จนถึง 100/100
- [เทสจากการ์ด](feedback_tests_derive_from_the_card_not_the_code.md) — การ์ด=SSOT · ห้ามแก้ expected ให้ตรงโค้ด
- [Comment การ์ด](feedback_comment_jira_card_when_loop_closes.md) — ครบ loop แล้วคอมเมนต์ Jira + ย้ายสถานะตาม MR (shipping.md §15)
- [Urgent thread](feedback_keep_urgent_thread_visible.md) — หัวข้อใหม่อย่ากลบงานด่วนที่ยังเสียหายอยู่
- [กวาด branch Codex แล้วส่งทั้งก้อน](feedback_sweep_codex_branches_and_ship_whole.md) — 10-11 release branch ของ Codex เป็น superset ของ branch ผม · "BLOCKED รอคนยืนยัน" ใน ledger ของ Codex ไม่ใช่ blocker ของ PO
- [auto mode บล็อกเปิด MR](reference_auto_mode_blocks_glab_mr_create.md) — เตรียม JSON + create-mrs.sh ให้ผู้ใช้กดเอง · project id chat-core 849 / agent 850 / admin-fe 851 / rag-core 787 / connector 882

- [OC2Plus develop↔main แยกสาย](reference_oc2plus_backoffice_develop_main_diverged_2026_10_11.md) — 10-11 ทั้ง 5 repo main merge เข้า develop ไม่ได้ (44/22/25/40/14 conflicts) · OC-4736 รวมสายครั้งเดียว · twin คือ workaround เดิม

- [เทียบผลคำสั่งสองฝั่งด้วย rtk proxy](feedback_compare_command_outputs_with_rtk_proxy.md) — rtk ตัดผล go vet เหลือ "… 19 more lines" ทำให้ diff เพี้ยน · อ่านดิบทั้งสองฝั่งก่อนเทียบ
- [เช็ค env เองได้ ผู้ใช้ไม่รู้](feedback_check_read_only_env_facts_yourself_user_does_not_know.md) — 10-11 ถาม "dev มี permission ไหม" แล้วผู้ใช้ตอบไม่ได้ · teleport login อยู่ แค่ port-forward rps + grpcurl GET · อย่าถามคนที่ไม่รู้
- [nunt-mode profile](project_nunt_mode_agent_profile.md) — 2026-10-12 สร้างด้วย /my-mode: 2 ชั้น (tier 0-4 + สไตล์ 29 ข้อ) · eval ครั้งแรก REVISE · ยังไม่แชร์ทีมจนกว่า re-eval tier 2
- [Robot ตัวแปรไม่แยกตัวพิมพ์](reference_robot_variables_are_case_insensitive_loop_var_overwrites_suite_var.md) — ${m} ใน FOR ทับ ${M} ของ suite เงียบ ๆ · ชนมา 3 ครั้ง · counter อยู่ใต้ auth_guard:*
## Jira
- [Sweep ผ่านไฟล์](reference_jira_board_sweep_via_saved_results.md) — ผลใหญ่เซฟเป็นไฟล์ parse ด้วย python
- [BOLA](reference_bola_jira_project.md) · [OC2Plus](reference_oc2plus_jira_project.md) · [Patona](reference_pat_jira_project.md) · [Sprint ids](reference_jira_sprint_ids_not_contiguous.md) · [PAT sprints](reference_pat_board_sprints.md) · [PAT epics](project_pat_epic_links_unwired.md) · [Crosses sessions](reference_jira_mcp_crosses_responses_between_sessions.md) · [Quirks](reference_jira_mcp_search_quirks.md) · [ADF Thai](reference_jira_editissue_adf_breakage.md) · [No fallback](reference_no_local_jira_fallback.md) · [403 writes](reference_jira_mcp_writes_403_midsession.md)

## Tests
- [slog.Init ทับทั้ง binary](reference_slog_init_once_silences_the_test_binary.md) — sync.Once · init() ที่ปิดเสียงทำให้เทสที่อ่าน log จริง fail เหมือนโค้ดพัง
- [ssk user-detail 2 แถว](reference_ssk_widget_user_detail_height_is_a_string.md) — widgetHeight ต้องเป็น string "6" · สูงสุด 7 แถว
- [Robot CI ไร้ target](reference_sukispace_robot_ci_has_no_target_service.md) — sukispace api-automate ยิง localhost:8080 ที่ไม่มี service → แดงเสมอ
- [Uncollected](reference_green_count_hides_uncollected_suite.md) · [Sentry dual](reference_sentry_dual_hub_on_version_skew.md) · [Stub loose](reference_test_stub_more_permissive_than_service.md) · [testify](reference_testify_permissive_default_wins.md) · [Timing](reference_timing_dependent_concurrency_tests.md) · [Turbo](reference_turbo_cache_crosssession_false_green.md) · [Lit SSR](reference_lit_react_node_condition_hollows_tests.md) · [DS testId](reference_ds_testid_is_a_property.md) · [jsdom 25](reference_node25_localstorage_jsdom_conflict.md)

## Go / DB
- [ns octoplus ไม่ใช่ sellsuki](reference_oc2plus_cluster_namespace_is_octoplus.md) — internal host เทส scope ได้ · gateway จริง crmapi.staging-th.oc2.plus ทับ header ปลอม
- [Files≠schema](reference_migration_files_are_not_applied_schema.md) · [envDefault](reference_envdefault_localhost_masks_missing_config.md) · [goqu](reference_goqu_dialect_blank_import.md) · [AutoMigrate](reference_gorm_pgx_libpq_automigrate.md) · [Drops false](reference_gorm_updates_drops_false.md) · [Lease](reference_lease_claim_ownership_bug_class.md) · [Partial idx](reference_pg_partial_index_onconflict_generic_plan.md) · [404 open](reference_ambiguous_404_fail_open.md) · [Own guard](reference_auth_endpoint_behind_own_guard.md) · [Kafka silent](reference_kafka_silent_publish_failure.md) · [Fiber ctx](reference_fiber_ctx_strings_corrupt_prometheus_labels.md)

## AI chat / KB (current thread)
- [KB100 eval 55/100 + ทิศทาง RAG](project_kb100_eval_state_and_rag_direction.md) — 2026-10-06 ตารางคือทั้งหมดที่หาย · เพดาน 8000 ไบต์ · เลิก chunk RAG → อ่านทั้งเอกสาร
- [local kb_rag_local DB](reference_local_kb_rag_db_and_milvus.md) — docker exec, canonical_knowledge_document, chunk text อยู่ใน Milvus
- [Drive MCP เปิดด้วย ID](reference_google_drive_mcp_shared_files_by_id.md) — search ไฟล์ที่คนอื่นแชร์ได้ {} · text export ไทยสระหาย
- [AI chat staging = ns sellsuki](reference_ai_chat_staging_is_ns_sellsuki_and_how_to_release_there.md) — dev-th ไม่มี stack · release ด้วยมือ: migration Job + manual deploy job
- [inbound forward local](reference_chat_core_local_inbound_forward_for_live_tests.md) — ยิงข้อความลูกค้าเข้า chat-core local ตรง ๆ · อ่านผลจาก messages/decision_traces/fallback_cases
- [connector 403 = token chat-core ไม่ตรง](reference_connector_403_is_often_chat_core_token_mismatch.md) — CHAT_CORE_SERVICE_TOKEN ≠ chat-core SERVICE_TOKEN → 401 กลายเป็น 403 เงียบ ๆ · พิสูจน์ด้วย pod ใน cluster
- [แอป FB staging = test app](reference_staging_fb_app_is_a_test_app.md) — 1722666062152698 เป็น test app ของ Sellsuki-New · เปิด Live ไม่ได้ · ตอบเฉพาะคนที่มี role · โดเมน staging ตั้งครบแล้ว 10-12
- [Sellsuki-New ยังไม่ผ่าน review](reference_sellsuki_new_meta_app_state.md) — Live แล้วแต่ pages_messaging = Standard → ตอบได้เฉพาะคนมี role · webhook ชี้ tunnel local bearyweb · แชร์ staging+prod ต้องมี relay
- [เปลี่ยนโมเดล = สิทธิ์ provider](project_model_change_is_provider_level.md) — 2026-10-12 PO: sellsuki.provider.llm.update เท่านั้น · company admin แก้ prompt ได้ · document mode ยังเป็น company admin
- [poppler 25.03 ทำไทยใน PDF แตก](reference_poppler_2503_breaks_thai_pdf_text.md) — image python:3.12-slim = 25.03 · Mac = 26 · local ดูดีเพราะรุ่น poppler ไม่ใช่โค้ด · แก้ data-pipeline !22
- [QUOTA_EXHAUSTED = OpenRouter 402](reference_ai_agent_quota_exhausted_is_openrouter_402.md) — เช็ก /api/v1/credits ก่อนเชื่อผลรัน
- [monorepo docs อยู่ AI-49](reference_monorepo_docs_live_on_ai49_branch.md) — origin/main ค้าง ส.ค. · ledger commit ลง fix/AI-49 · อย่า push แทน session อื่น
- [control-tower แยกทาง main↔AI-49](reference_control_tower_data_js_diverged_main_vs_ai49.md) — ยังไม่เคาะตัวจริง (MR !29)

- [System testing direction](project_system_testing_direction_2026_10_11.md) — 2026-10-11 เคาะ 4 ข้อ: invariants suite ข้ามบริการเป็น gate local · seed-scenario.sh · permission ratchet ใน ship-check · doctor/dev-ensure + local-stack state · verifier บังคับก่อน MR
## Sub-indexes — open when the topic comes up
- [Identity / CCS / rps + local-stack traps](index_identity_ccs.md) — Kratos, Keto, rps, CCS config, local QA/login/Docker/overmind traps
- [BOLA](index_bola.md) — deploy topology, access model, LINE/LIFF, contact, APM
- [OC2Plus](index_oc2plus.md) — prod DB state, member app, 3rd-party API, loyalty, consent, LINE setup, product decisions
- [Personal / machine / infra](index_infra.md) — machine, local stack, overmind, Caddy, rtk, DNS, CI runners, clusters
- [Git / CI](index_git_ci.md) — merge traps, submodules, GitLab CI, review bot, helm, glab · [auto-merge ไม่รอ CI](reference_glab_auto_merge_merges_immediately_without_ci_gate.md) · [Outdated deploy job](reference_gitlab_outdated_deployment_job_skips_the_newer_pipeline.md)
- [Go CLI verify TLS ไม่ได้ใน sandbox](reference_sandbox_blocks_go_cli_tls_and_ssh.md) — glab/tsh/kubectl ตาย OSStatus -26276 · SSH ก็บล็อก · ใช้ GitLab REST ผ่าน curl ทำ branch/commit/cherry-pick/MR ได้ครบ
- [Patona / OMS / QMS / SukiPay / DS](index_patona_oms_ds.md) — Patona, OMS2, QMS, SukiPay, design system
- [AI Chat Platform](index_ai_chat.md) — plan, chat-core, rag-core, AI agent, SLA, FB/Messenger
- [Team workflow goal](project_team_workflow_sharing_goal.md) · [Test users = infra](project_test_users_as_infrastructure.md) · [pstack / Control Glass](project_pstack_as_workflow_benchmark.md) · [SukiSpace marketplace+AIS](project_sukispace_marketplace_ais_vision.md)
- [dev: user ใหม่ติด 2 จุด](project_dev_onboarding_dead_ends_2026_10_11.md) — admin FE สร้าง company ผ่าน route provider (403) ทั้งที่มี POST /v1/user/company · seed portal ชี้ crm.dev-th.oc2.plus (NXDOMAIN) ต้องเป็น crm.dev.oc2.plus · ยังไม่แก้
