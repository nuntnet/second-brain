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
- [Standing approval merge→deploy staging](feedback_standing_approval_merge_deploy_staging_after_gates.md) — 10-23 PO: เกตครบ (verifier A, ship-check, bot ไม่มี MUST FIX, pipeline เขียว) → merge/deploy staging/migration Job ได้เองแล้วรายงาน · main→prod ยังถาม
- [Comment การ์ด](feedback_comment_jira_card_when_loop_closes.md) — ครบ loop แล้วคอมเมนต์ Jira + ย้ายสถานะตาม MR (shipping.md §15)
- [Urgent thread](feedback_keep_urgent_thread_visible.md) — หัวข้อใหม่อย่ากลบงานด่วนที่ยังเสียหายอยู่
- [กวาด branch Codex แล้วส่งทั้งก้อน](feedback_sweep_codex_branches_and_ship_whole.md) — 10-11 release branch ของ Codex เป็น superset ของ branch ผม · "BLOCKED รอคนยืนยัน" ใน ledger ของ Codex ไม่ใช่ blocker ของ PO
- [Stacked promotion wave develop→main](reference_stacked_promotion_wave_develop_to_main.md) — 10-22 chain-merge ใน scratch worktree, conflict = เอาไฟล์ develop, ปิดด้วย sync branch ให้ main == develop · classifier ไม่ให้กด merge ต้องให้ผู้ใช้รันสคริปต์
- [auto mode บล็อกเปิด MR](reference_auto_mode_blocks_glab_mr_create.md) — เตรียม JSON + create-mrs.sh ให้ผู้ใช้กดเอง · project id chat-core 849 / agent 850 / admin-fe 851 / rag-core 787 / connector 882

- [OC2Plus develop↔main แยกสาย](reference_oc2plus_backoffice_develop_main_diverged_2026_10_11.md) — 10-11 ทั้ง 5 repo main merge เข้า develop ไม่ได้ (44/22/25/40/14 conflicts) · OC-4736 รวมสายครั้งเดียว · twin คือ workaround เดิม
- [rps ไม่ได้แยกสาย / mgmt-backend แยกจริง](reference_rps_and_management_backend_develop_main_diverged.md) — 10-20 rps: merge MS-1420 ลบ dir 0032/0033 ของ main (คืนด้วย !178) · mgmt-backend (path sellsuki/patona/): QMS port สองรอบ 61/52 → merge main→develop ครั้งเดียว, auto-merge ซ้อน 6 ไฟล์ จับด้วย go vet · push โดน classifier บล็อก · 10-22 main มีเทสห้ามเลขซ้ำ → seed เป็น 0035 บน main

- [rtk cat ตัด comment](feedback_rtk_cat_strips_comments_never_use_cat_to_rewrite_files.md) — 10-20 `cat x > f` ลบ comment 13 บรรทัดใน main.go · เขียนไฟล์ด้วย python bytes เท่านั้น
- [เทียบผลคำสั่งสองฝั่งด้วย rtk proxy](feedback_compare_command_outputs_with_rtk_proxy.md) — rtk ตัดผล go vet เหลือ "… 19 more lines" ทำให้ diff เพี้ยน · อ่านดิบทั้งสองฝั่งก่อนเทียบ
- [เช็ค env เองได้ ผู้ใช้ไม่รู้](feedback_check_read_only_env_facts_yourself_user_does_not_know.md) — 10-11 ถาม "dev มี permission ไหม" แล้วผู้ใช้ตอบไม่ได้ · teleport login อยู่ แค่ port-forward rps + grpcurl GET · อย่าถามคนที่ไม่รู้
- [nunt-mode profile](project_nunt_mode_agent_profile.md) — 2026-10-12 สร้างด้วย /my-mode: 2 ชั้น (tier 0-4 + สไตล์ 29 ข้อ) · eval ครั้งแรก REVISE · ยังไม่แชร์ทีมจนกว่า re-eval tier 2
- [Robot ตัวแปรไม่แยกตัวพิมพ์](reference_robot_variables_are_case_insensitive_loop_var_overwrites_suite_var.md) — ${m} ใน FOR ทับ ${M} ของ suite เงียบ ๆ · ชนมา 3 ครั้ง · counter อยู่ใต้ auth_guard:*
- [Tiers + Decision Inbox](project_workflow_tiers_and_inbox_2026_10_13.md) — 2026-10-13 AC ชั้น A/B/C (B ไม่บล็อก MR เข้า develop) + คิวคำถามลงไฟล์ · ยังไม่ eval/commit · ผู้ใช้ยืนยันว่าต้องรัน agent ขนานต่อ
- [OC-4739 file-service access backfill](project_oc4739_file_service_access_backfill.md) — 10-13/14 CCS สร้าง role ให้ identity ตอนสร้างบริษัท + backfill · staging 14,749 บริษัท รันเป็น pod แยก (arm) · เหลือ AC8 + CCS crash
- [อ่าน log Job ก่อนลบ](feedback_read_job_logs_before_deleting_the_job.md) — 10-15 _0026 ล้ม 14 แต่ลบ Job ก่อนอ่าน · gormigrate รันซ้ำเป็น no-op · เซฟ log ลงไฟล์ก่อนเสมอ
- [retry ตอนเช้าเป็นระลอก](feedback_morning_retries_in_waves_runner_cluster_is_12_nodes.md) — 10-21 ยิง retry 28 job รวดเดียว → runner 12 node เต็ม "Insufficient cpu" รอบสอง + deploy ตาย · รอบละ ≤6–8 อ่านครั้งเดียวก่อนรอบถัดไป · classifier ปฏิเสธ loop รวม ให้แยกต่อ MR · govulncheck แดงทุก MR = stdlib เลื่อน bump toolchain 3 จุด
- [replay เข้า chat-core staging ใน pod](reference_staging_chat_core_replay_recipe_in_pod_wget.md) — wget ใน pod ด้วย $SERVICE_TOKEN, ยิงชุด bubble ใน exec เดียว, อ่าน inbound_ai_work/traces จาก psql ใน datastore, metric turn_superseded
- [agent staging Vault token หมดอายุ = crash ตอน restart](reference_ai_agent_staging_vault_token_expiry_crash_on_restart.md) — 10-22 deploy ล้ม helm timeout จริง ๆ คือ Vault 403 ตอน boot · pod เก่ารอดจน restart · ดู logs --previous ก่อน retry · ต้องให้คนมีสิทธิ์ Vault ออก token ใหม่
- [กลางคืนปิดแค่ staging-th](reference_night_window_is_staging_th_only_production_runners_run.md) — Production runner (5125 มี tag amd) รับ job ตอนกลางคืน; job tag staging-th เท่านั้นที่รอเช้า · ดู runner.description ก่อนสรุป
- [scenario harness พิสูจน์ lead/goal/step บน local](reference_real_conversation_scenario_harness_lead_goal_step_2026_10_24.md) — scenario.py ลูกค้าคุยทีละเทิร์น · ผล 10-24: โค้ดมีแต่วงไม่ปิด (ไม่มีอะไรสร้าง lead, slot ตัวเลขเติมไม่ได้, converted 9/10 มั่ว, placeholder หลุดถึงลูกค้า) · local ไม่มี KB ต้องซ้ำบน staging
- [เปิด console local บน localhost ด้วย seed identity](reference_local_ai_chat_console_open_on_localhost_with_seed_identity.md) — overmind -l …,web-ai-chat-admin · http://localhost:5191 (vite proxy) · login curl ที่ :4433 แล้ววาง cookie ผ่านไฟล์ static · route /w/<ws>/settings/<page>
- [เทส console staging ใน pane](reference_staging_ai_chat_console_drive_recipe.md) — URL ai-chat-admin.staging-th, inbox = /w/<id>, PO ล็อกอินใน pane เอง, resize 1280 ก่อนถึงเห็น picker "รับเอง"
- [messaging-backend Vault 403 → FB webhook 500](reference_messaging_backend_staging_vault_token_403_makes_fb_webhook_500.md) — 10-24 ทักเพจ staging ไม่มี AI ตอบ · 500 ใน 4ms ไม่มี log · พิสูจน์ lookup-self ในพอด · ต้องคนมีสิทธิ์ Vault ออก token
- [ถามตัดสินใจต้องมีบริบทครบ](feedback_ask_decisions_with_full_context_not_bare_lists.md) — 10-24 PO ว่า 'ถามสั้นๆ ไม่รู้เรื่อง' · คืออะไร/ทำไม/ทางเลือกพร้อมผล/ข้อแนะนำ/ตอบแบบไหน · อ่านการ์ดต้นทางก่อนอธิบาย
- [แผน Playbook/Goal/Lead/KB → การ์ด AI-312..327](project_playbook_goal_lead_kb_cards_2026_10_24.md) — 10-24 ค่ำ เลขการ์ดต่อรหัสร่าง · คำตัดสิน PO · ของที่ยังค้าง · เอกสารหลัก docs/ai-chat-playbook-goal-lead-kb-design-2026-10-24.md
- [Sprint sweep = deploy ไม่ใช่พิสูจน์ AC](feedback_sprint_sweep_board_column_means_deploy_not_ac_proof.md) — 10-14 RTT(staging)=โค้ดบน main ทุก repo+deploy · คอขวด OC2Plus คือ main twin ค้าง ไม่ใช่ PO · merge-wave.py · Jira ทีละคอล
- [CI รอบแรกของ spec ที่ไม่เคยรัน](reference_first_ci_run_of_never_run_playwright_specs_finds_real_bugs_and_axe_baseline_drift.md) — 10-24 ai-chat-admin FE: testId DS ไม่ใช่ attribute, ssk-button ชน 2 node, route swap ทำ header หลุด, ฟอนต์ไทย CI ต่าง, axe baseline ต่าง · ห้าม rebase branch ที่ push แล้ว (merge main) · main CI ต้อง staging-th ไม่รัน 21-09
- [Wave 1 state 10-25](project_wave1_playbook_goal_lead_kb_state_2026_10_25.md) — โค้ด 8 การ์ด push แล้ว ไม่มี MR/verifier · D-029..D-032 · kit ต้อง tag · บทเรียนบรีฟ agent
- [sandbox บล็อก localhost → DB test skip เงียบ](reference_sandbox_blocks_localhost_so_db_tests_skip_silently.md) — นับ --- PASS/SKIP จาก -v ไม่ใช่ exit code · worktree sparse · modfile ชั่วคราว
- [pgx cached plan หลัง ADD COLUMN](reference_pgx_cached_plan_breaks_old_pods_after_add_column_migration.md) — pod เก่า error จน restart · migrate+roll ช่วงเดียว · PreferSimpleProtocol
- [ดิสก์เต็ม → Docker พัง](reference_disk_full_kills_docker_desktop_go_cache_is_the_first_thing_to_delete.md) — 10-25 go cache 19 GB + Docker.raw 100 GB · ลบ cache ที่ agent สร้างเอง · kill backend ที่ค้าง · kafka ต้อง start เอง · chat-core ไม่ restart เอง (เขียนไฟล์ไบต์เดิมให้ air เห็น) · ใช้ GOCACHE เดียวแล้วลบ
- [AI-318 ผลวัดขึ้นกับโมเดล](project_ai318_interest_gate_depends_on_the_model_2026_10_25.md) — gemini 0.708/0.810 ไม่ผ่าน · gpt-4o-mini 0.952/0.952 ผ่าน · ชุดทดสอบเป็นของผู้เขียนโค้ด อย่าปรับ prompt กับชุดเดิม · D-038
- [Oc2plus SaaS plan epic OC-4763](project_oc2plus_saas_plan_epic_oc4763_decisions_2026_10_16.md) — 2026-10-16 PO: quota soft ค่าเริ่มต้น · ไม่ตั้ง active_until · billing_start แยกนอก QMS · ตั้งโควตาผ่าน API · แคตตาล็อกเฉพาะ sellsuki · ตัวเลขทั้งหมดเป็นค่าเสนอรอ BD
## Jira
- [Sweep ผ่านไฟล์](reference_jira_board_sweep_via_saved_results.md) — ผลใหญ่เซฟเป็นไฟล์ parse ด้วย python
- [BOLA](reference_bola_jira_project.md) · [OC2Plus](reference_oc2plus_jira_project.md) · [Patona](reference_pat_jira_project.md) · [Sprint ids](reference_jira_sprint_ids_not_contiguous.md) · [PAT sprints](reference_pat_board_sprints.md) · [PAT epics](project_pat_epic_links_unwired.md) · [Crosses sessions](reference_jira_mcp_crosses_responses_between_sessions.md) · [Quirks](reference_jira_mcp_search_quirks.md) · [ADF Thai](reference_jira_editissue_adf_breakage.md) · [No fallback](reference_no_local_jira_fallback.md) · [403 writes](reference_jira_mcp_writes_403_midsession.md)

## Tests
- [slog.Init ทับทั้ง binary](reference_slog_init_once_silences_the_test_binary.md) — sync.Once · init() ที่ปิดเสียงทำให้เทสที่อ่าน log จริง fail เหมือนโค้ดพัง
- [ssk user-detail 2 แถว](reference_ssk_widget_user_detail_height_is_a_string.md) — widgetHeight ต้องเป็น string "6" · สูงสุด 7 แถว
- [Robot CI ไร้ target](reference_sukispace_robot_ci_has_no_target_service.md) — sukispace api-automate ยิง localhost:8080 ที่ไม่มี service → แดงเสมอ
- [Uncollected](reference_green_count_hides_uncollected_suite.md) · [Sentry dual](reference_sentry_dual_hub_on_version_skew.md) · [Stub loose](reference_test_stub_more_permissive_than_service.md) · [testify](reference_testify_permissive_default_wins.md) · [Timing](reference_timing_dependent_concurrency_tests.md) · [Turbo](reference_turbo_cache_crosssession_false_green.md) · [Lit SSR](reference_lit_react_node_condition_hollows_tests.md) · [DS testId](reference_ds_testid_is_a_property.md) · [jsdom 25](reference_node25_localstorage_jsdom_conflict.md)
- [AI chat permission staging 10-27](project_ai_chat_staging_permission_state_2026_10_27.md) — Owner preset 3 generations (none / legacy / manage) · no preset holds usage/billing · switcher lists workspaces the user cannot open · how to check rps/Keto/pg18
- [มหานครโลหะ: การตัดสินใจเบี่ยง Proposal](project_mahanakhon_proposal_vs_roadmap_decisions_2026_10_14.md) — 2026-10-14 CRM เป็นเจ้าของ active/inactive+tier (Scrapyard รับ webhook) · แยกตาม company · เดือนเกิด/manual/QR ลูกค้าขอในที่ประชุม · ช่องว่าง: OC-4428 ไม่มี tier webhook, OC-2261 นอก roadmap

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
- [resume subagent ที่ตาย](reference_resume_dead_subagent_by_sendmessage_not_respawn.md) — API ENOTFOUND/spend limit → SendMessage ไป agentId เดิม ไม่ spawn ใหม่ · commit บน branch รอดแม้ scratchpad worktree หาย
- [Go CLI verify TLS ไม่ได้ใน sandbox](reference_sandbox_blocks_go_cli_tls_and_ssh.md) — glab/tsh/kubectl ตาย OSStatus -26276 · SSH บล็อก · git fetch/push ผ่าน HTTPS+token ได้ถ้า override insteadOf (10-17) · REST ผ่าน /usr/bin/curl
- [Patona / OMS / QMS / SukiPay / DS](index_patona_oms_ds.md) — Patona, OMS2, QMS, SukiPay, design system
- [AI Chat Platform](index_ai_chat.md) — plan, chat-core, rag-core, AI agent, SLA, FB/Messenger
- [Team workflow goal](project_team_workflow_sharing_goal.md) · [Test users = infra](project_test_users_as_infrastructure.md) · [pstack / Control Glass](project_pstack_as_workflow_benchmark.md) · [SukiSpace marketplace+AIS](project_sukispace_marketplace_ais_vision.md)
- [dev: user ใหม่ติด 2 จุด](project_dev_onboarding_dead_ends_2026_10_11.md) — admin FE สร้าง company ผ่าน route provider (403) ทั้งที่มี POST /v1/user/company · seed portal ชี้ crm.dev-th.oc2.plus (NXDOMAIN) ต้องเป็น crm.dev.oc2.plus · ยังไม่แก้
- [BOLA Postgres tests] (reference_bola_postgres_tests_need_disposable_container_and_two_dsn_forms.md) — throwaway postgres :55432 · DSN เป็น URL ใน contact_profile_repository แต่ key=value ใน contact_metrics_repository · race test จับแบบสุ่ม ต้องวนรัน
- [BOLA Jira ไม่มี In Review + merge ถูกบล็อก](reference_bola_jira_has_no_in_review_status_and_auto_mode_denies_merge.md) — ห้ามเดา status · ให้ผู้ใช้กด merge MR เอง
- [รีวิว mock ต้องเทียบ adapter จริง](feedback_mock_mode_ui_review_check_real_adapter_before_calling_defect.md) — 10-23 enum ดิบ/ชื่อ •••/ข้อความไทยใน EN ที่เห็นใน mock เป็นของ mock ทั้ง 3 ข้อ · grep real adapter ก่อนเรียก defect
- [console mock :5192](reference_ai_chat_admin_mock_preview_5192_no_login.md) — preview_start web-ai-chat-admin-mock ไม่ต้อง login · ws_demo_default · ดู layout/contrast ได้ ไม่ใช่ข้อมูล
- [Staging verifier preflight](reference_staging_verifier_preflight_tsh_and_pane_login.md) — 10-24 tsh staging login + pane login must be checked first; night/expired = zero verdicts; BOLA-311 join/CCS not on main
- [AI chat MR + Jira landing recipe](reference_ai_chat_mr_and_jira_landing_recipe_2026_10_25.md) — 10-25 token ต้องเอาจาก block gitlab.sellsuki.com · project id ทั้ง 6 repo · มีแค่ messaging มี develop (ที่เหลือเข้า main) · Jira In Review = transition 31
- [Vault/prod: ส่งคำสั่งให้ผู้ใช้รัน](user_has_vault_and_prod_access_agent_hands_over_commands.md) — nunt มีสิทธิ์ Vault+prod · agent แค่ให้คำสั่งครบ ไม่ต้องถามว่าใครมีสิทธิ์ · tsh หมดอายุ = ให้ login ก่อน
- [staging 503 = pg18 disk full](reference_staging_chat_core_503_is_often_shared_postgres_pg18_disk_full.md) — 10-26 pod 0/1 ไม่ crash → logs ต่อ DB ไม่ได้ → pg18-0 CrashLoop "No space left on device" (pg_wal) · ห้ามลบ WAL · เรื่องของเจ้าของ datastore
