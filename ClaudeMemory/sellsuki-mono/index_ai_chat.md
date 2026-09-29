---
name: index_ai_chat
description: "Sub-index of memories for AI Chat Platform — moved out of MEMORY.md 2026-09-27 to keep the loaded index under its size limit"
metadata:
  type: reference
---

# AI Chat Platform — sub-index

Open the topic file before relying on any line here.

- [Plan](project_ai_chat_platform_plan.md) · [Arch](reference_ai_platform_architecture_artifact.md) · [E0](project_ai_chatsystem_e0_structure.md) · [Gating](project_ai_platform_deploy_gating.md) · [FE design](reference_ai_chat_frontend_design.md) · [MVP](project_ai_mvp_integration.md) · [Neutral](project_chat_platform_is_vertical_neutral.md) · [Sprint 2-4](project_ai_sprint234_autonomous_run.md) · [Merge order](project_ai_chatcore_merge_order.md) · [Topology risk](project_ai_merge_topology_risk.md)
- [SLA](project_sla_ladder_engine_state.md) · [Board stale](reference_ai_board_stale_cards.md) · [In Review](reference_ai_board_in_review_means_merged_unverified.md) · [Placeholder](project_ai_placeholder_cards_review.md) · [Gap sweep](project_ai_backlog_gap_sweep_202608.md) · [E8](project_e8_remaining_blockers.md) · [Audit](reference_ai_chatbot_completeness_audit.md) · [Stateless](project_ai_agent_stateless.md) · [agent CI](reference_ai_agent_ci_gaps.md) · [Flag](reference_flag_without_enforcement.md) · [2658](project_pat2658_reference_collision.md)
- [Admin BFF](reference_chatcore_is_the_admin_bff.md) · [Bootstrap](project_chatcore_role_bootstrap_and_eastwest_auth.md) · [Unprefixed](reference_chatcore_unprefixed_chat_workspace.md) · [List derived](project_chatcore_company_list_derived_not_asked.md) · [0066](reference_chatcore_migrations_break_at_0066.md) · [Route timeout](reference_chatcore_route_tests_timeout_under_load.md) · [WS flaky](reference_chatcore_route_workspace_flaky_under_load.md) · [Zero panic](reference_chatcore_zero_value_usecase_panics.md) · [core CI](reference_chatcore_ci_gaps.md)
- [ของมีอยู่ แต่ไปไม่ถึง](reference_it_exists_but_cannot_reach_the_cluster.md) — สี่ครั้งในคืนเดียว · ตรวจของในคลัสเตอร์ ไม่ใช่ job ที่ผลิตมัน
- [Boot guard pins image](reference_boot_guard_pins_an_old_image_silently.md) — cluster เขียว 1/1 แต่ image ค้าง 16 วัน เพราะ guard ตอน boot + Helm --atomic
- [2 hops](reference_chatcore_ragcore_two_hops.md) — console KB hop กับ reply retrieval hop คนละ credential ตันคนละเรื่อง
- [Dual embed](project_ragcore_dual_embedding_paths.md) · [Tiers](reference_rag_core_visibility_tiers.md) · [KB blockers](reference_kb_entries_three_blockers.md) · [Conv Intel](project_ai_conversation_intelligence.md) · [Port map](project_ai_admin_port_backend_map.md) · [No FE](reference_backend_without_frontend_is_invisible.md) · [150](project_ai150_members_read_only.md) · [119](project_ai119_push_deferred.md) · [115](project_ai115_conversation_goal_backend_only.md) · [146](project_ai146_onboarding_wizard_not_started.md)
- [case_type](project_case_type_setting_has_no_home.md) · [Provider=CCS1](project_provider_create_lives_in_ccs1.md) · [lang=th](project_preferred_language_is_constant_th.md) · [Fact vocab](project_fact_vocabulary_collision.md) · [96](project_ai96_template_catalog_reality.md) · [16](project_ai16_field_set_admin_configurable.md) · [125](project_ai125_oauth_long_lived_server_side.md) · [33 WS](project_ai33_websocket_is_required.md) · [F08](project_f08_fact_schema_versus_values.md) · [Stuck job](reference_stuck_ci_job_holds_resource_group.md)
- [FB subscribe](reference_fb_page_must_be_subscribed_to_app.md) · [FB diagnose](reference_fb_page_delivery_diagnosis_endpoint.md) · [ack+drop](reference_webhook_ack200_and_drop_looks_like_no_delivery.md) · [No staging](reference_ai_chat_has_no_staging_deployment.md) · [Procfile kills](reference_procfile_messaging_disables_chat_module.md) · [4 env keys](reference_ai_chat_local_env_keys_missing.md) · [No Kratos URL](reference_chatcore_missing_kratos_url_503.md)
- [Kratos mail](reference_local_kratos_mail_mailslurper.md) — OTP/verify เมล local อยู่ Mailslurper UI :4436 API :4437 (compose ป้ายสลับ) · curl ต้อง rtk proxy
- [kratos-ui local branch](reference_kratos_ui_local_needs_feature_branch.md) — อย่าสลับ local ไป develop ตรงๆ: signup พัง (DISABLE_CONSENT) + kratos.yml บน develop เก่า v1.3 · อ่าน develop ด้วย git archive
- [signup ข้าม Kratos](reference_kratos_ui_registration_bypasses_kratos_policy.md) — สมัครผ่าน admin CreateIdentity: รหัส "123" ผ่าน, ไม่เช็ค CSRF, OIDC ซ้ำไม่ link · โค้ดใหม่ทำโค้ดเก่าตาย, 5 ครั้งผิด flow ตาย
- [OTP dev→staging](reference_member_api_dev_otp_hits_staging_messaging.md) — member-api dev ยิง messaging ของ staging (.sellsuki ไม่ใช่ .sellsuki-dev) · ร้าน dev ได้ OTP 503 action_not_configured ทั้งที่ตั้งค่าครบ
- [Auth UX PAT-2730..34](project_auth_ux_card_cluster_pat2730.md) — การ์ด kratos-ui ห้าใบ + PO เคาะ: หน้าจอเป็นกลาง บอกวิธีเข้าเฉพาะในอีเมล (ไม่ทำ identifier-first)
- [OC2Plus MCP](project_oc2plus_mcp_assistant_idea.md) — OC-4625 live on dev 2026-09-28 · 23 tool อ่านอย่างเดียวใน backoffice-api /mcp · Hydra client dev เท่านั้น · prod ติด DPA · หน้า "เชื่อมต่อ AI" AC-C6
- [Oathkeeper/Hydra บน cluster](reference_oathkeeper_hydra_cluster_state.md) — rule เป็น CRD ไม่อยู่ใน repo · backoffice รับแค่ cookie · rag-core-mcp ใช้ Hydra introspection แล้ว · octoplus ไม่มี NetworkPolicy
- [rag-core MCP มีอยู่แล้ว](reference_rag_core_mcp_prior_art.md) — repo sellsuki-rag/rag-core (ไม่ใช่ poc ในเวิร์กสเปซ) · kimzey/PAT-2691 · switch_company, OAuth ผ่าน Claude/Gemini/Codex
- [MCP eval ผ่าน claude -p](reference_claude_code_mcp_headless_eval_traps.md) — อย่าใช้ config ชื่อ server ซ้ำระหว่างรัน (credential หาย) · รันทีละข้อ · ผู้ใช้หลายบริษัท model เรียก whoami ก่อนตามออกแบบ วัด tool หลัง whoami
