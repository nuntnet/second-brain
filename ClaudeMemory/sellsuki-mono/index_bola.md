# BOLA — sub-index

Moved out of MEMORY.md on 2026-10-11. One line per memory; open the file before relying on it.

- [BOLA v2 cleanup goal](project_bola_v2_cleanup_goal_2026_10_12.md) — 2026-10-12 เคลียร์การ์ดเก่าก่อน v2 · S8/S9/S10 จัดแล้ว · เหลือเฟส rewrite · [Jira workflow+bulk ผ่าน Chrome](reference_bola_jira_workflow_and_bulk_writes_via_chrome.md) — transition id, sprint id, resolution, classifier ต้องให้ user อนุมัติก่อนปิดเยอะ
- [WMMT-1634 LINE แจ้งผู้แจ้ง BPM](project_wmmt1634_bpm_line_notify_liff_direction.md) — เสนอ LIFF+ID token ผูกเรื่อง + Jira Automation→APM · ยังไม่เคาะ
- [BOLA dev DB อ่านไม่ได้เฉยๆ](reference_bola_dev_db_not_readable_without_creds_or_client.md) — kubectl ใช้ได้แต่ไม่มี psql/ไม่มี read endpoint บน system token · เช็คผ่านหน้า contact ใน BOLA console (resolver เดียวกับกฎเมนู)
- [gitignore บัง cmd/bola_server](reference_bola_backend_gitignore_hides_cmd_bola_server.md) — `git add` ข้ามไฟล์ใหม่เงียบ ๆ
- [Oathkeeper global=staging](reference_oathkeeper_global_cookie_session_is_staging.md) — rule dev ต้องระบุ config เอง
- [SRE GitOps repos](reference_sre_gitops_repos_and_sync_policy.md) — api-gateway/bridge dev ปิด auto-sync
- [BOLA LLM fail-closed](reference_bola_llm_metering_is_fail_closed_dev_borrows_staging_agent.md) — ไม่มี AI_PLATFORM_KIT_* = ตอบไม่ได้
- [Deeplink=ไม่ใช่ kratos build](reference_bola_deeplink_single_workspace_means_not_kratos_build.md) — bundle dev เป็น local_jwt
- [LIFF error ladder](reference_bola_liff_shell_error_ladder.md) — ข้อความบนจอบอกสาเหตุอยู่แล้ว
- [member→BOLA ไม่เคยเชื่อม](reference_member_api_to_bola_link_never_worked.md) — BOLA_API_KEY ต้องคนตั้งเอง
- [LINE login OA check](reference_line_login_oa_check_was_a_never_loaded_field.md) · [LIFF endpoint เดียว](reference_liff_single_endpoint_flow_in_link_not_endpoint.md) · [OA ซ้ำข้าม workspace](reference_bola_line_oa_channel_is_globally_unique.md) · [backfill kratos](reference_bola_backfill_kratos_ids_cannot_run_as_documented.md)
- [dev is live](reference_bola_dev_env_exists_and_is_live.md) · [Staging gate](reference_manual_staging_gate_silent_drift.md) · [CCS unwired](project_ccs_bola_provisioning_unwired.md) · [Topology](project_bola_deploy_topology.md) · [values](project_bola_deploy_values_in_repo.md) · [Kratos gap](project_bola_saas_kratos_deploy_gap.md) · [Boot migrate](project_bola_migrations_jsonb.md) · [Loki](reference_bola_staging_loki.md)
- [§6a invite](project_bola309_invite_via_ccs.md) · [§6a lane2](project_bola309_lane2_add_existing_member.md) · [Access](project_bola_saas_access_model.md) · [auth mode](project_bola_auth_mode_deployment.md) · [Keto direct](project_bola_rbac_keto_direct.md) · [ops=CCS1](project_bola_ops_visibility_ccs1.md) · [SSO staging](project_bola_kratos_sso_staging.md) · [Follower ×2](reference_bola_follower_metadata_two_stores.md)
- [Contact](project_bola_contact_profile_model.md) · [Upsert](reference_bola_contacts_upsert_api.md) · [int vs bool](project_bola_is_enabled_int_bool_mismatch.md) · [Segment](project_segment_export_static_snapshot.md) · [Scoping](project_bola_workspace_scoping_bugs.md) · [293 chain](project_bola293_chain_state.md) · [FB dev-mode](project_fb_page_dev_mode_gate.md) · [Reply token](project_bola_reply_token_epic.md) · [Personalization](project_bola_ai_chatbot_personalization.md) · [APM hook](project_bola_apm_webhook_design.md) · [APM batch](project_apm_scheduled_batch_epic.md)
- [Follower IdP epic BOLA-337](project_bola_follower_idp_epic.md) — 2026-10-11 ผู้ติดตามยืนยันตัวตนกับ IdP ต่อ OA · สิทธิ์จาก claims + แอดมิน · การ์ด 338/339/340/341
- [Prod deploy = tag + approve ทุก job · QMS prod ล้าสมัย](reference_bola_prod_deploy_mechanics_and_gaps_2026_10_22.md) — 10-22 helm --atomic, secretKeyRef ไม่ optional, migration ตอน boot, prod ค้าง v1.0.29/v1.0.23 ตั้งแต่ ส.ค., QMS prod v0.1.1 ไม่มี Reserve/Commit RPC
