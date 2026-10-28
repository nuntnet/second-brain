# Identity / CCS / rps / local-stack traps — sub-index

Moved out of MEMORY.md on 2026-10-11 to keep the root index small. One line per memory; open the file before relying on it.

## Identity / CCS / rps
- [rps cache ไม่เคยเปิด](reference_rps_permission_cache_is_never_enabled.md) — ถอนสิทธิ์มีผลทันทีทุก env
- [preset ถูก copy](reference_preset_copy_vs_shared_role.md) — AI-252 (2026-09-28) CCS develop มี reconciler แล้ว
- [dev ไร้ system admin](reference_dev_has_no_provider_owner_role_or_system_admin.md) — seed ผ่าน rps gRPC เท่านั้น
- [return_to list](reference_kratos_return_to_allowlist_location.md) · [Not signin](reference_kratos_identity_exists_is_not_registered.md) · [SPOF](reference_shared_identity_infra_singleton.md) · [Hardcode loop](reference_local_identity_hardcode_loop.md) · [Kratos debug](reference_local_kratos_identity_debugging.md) · [i18n seed](reference_local_i18n_config_seeding.md) · [Config ns](reference_ccs_config_namespaces.md) · [AI ns](reference_ccs_ai_chat_config_namespace.md) · [Topology](reference_ccs_env_topology.md) · [CCS3 FE](reference_ccs3_frontend_facts.md)
- [Global gate](reference_ccs_global_config_permission_gate.md) · [Module broken](reference_ccs_go_module_path_broken.md) · [values in repo](reference_ccs_deployment_values_live_in_the_repo.md) · [QA pact](project_ccs3_qa_change_notification_agreement.md) · [userId defaults](reference_ccs_config_userid_silently_returns_schema_defaults.md) · [Keto≠rps](reference_keto_and_rps_catalog_disagree.md) · [Grant perm](reference_rps_grant_permission_to_role.md) · [Invite burns](reference_invite_accept_burns_the_code_on_grant_failure.md)
- [membertier ไม่มีบน dev](reference_dev_rps_membertier_missing_0025_present.md) — auto mode บล็อก grant สิทธิ์ rps
- [เจ้าของบริษัท](reference_rps_company_owner_lookup.md) — /internal/v1/role-assignments · role `Company Owner`
- [proto ×9](reference_rps_proto_is_vendored_per_consumer.md) · [internal API](reference_rps_internal_endpoints_confirmed.md) · [ListRoles](reference_rps_listroles_pointer_pagination.md) · [Keto staging](reference_keto_staging_permission_lookup.md) · [Presets](project_ccs_role_presets_apply_only_at_creation.md) · [rps mainline](reference_rps_dual_mainline.md) · [is_system](reference_rps_is_system_role_trap.md) · [kind prefix](reference_rps_identity_kind_must_be_prefixed.md) · [Assigned](reference_rps_list_assigned_roles_reverse_lookup.md) · [Underscore](reference_permission_code_underscore_rejected_before_query.md)
- [CCS1 revert trap](reference_ccs1_main_tree_revert_trap.md) — main ถูก revert ทั้ง tree 2026-09-29 (AKI-4098)
- [provider 500 ไร้ประเทศ](reference_ccs_provider_view_500_without_country.md) — แก้แล้ว !382/!383 dev/staging ยังไม่เช็ก
- [space-go ไม่มี gateway route](reference_space_go_has_no_gateway_route_mb_kind_forced_user.md) — SG-44: ไม่มี rule ใน git ทุก env · MB บังคับ X-User-Kind=sellsuki.user · JWK plaintext ใน ory-helm
- [config ขาดบน develop](reference_central_config_develop_lacks_main_seeds.md) — seed อยู่ main อย่างเดียว · ซิงก์ !149/!381
- [Tenant kinds](reference_entity_lib_tenant_kinds.md) · [Gen churn](reference_permission_generator_nondeterministic.md) · [file-svc kind](reference_file_service_keto_subject_kind.md) · [Audit enum](reference_audit_action_is_closed_enum.md) · [Audit targets](reference_audit_log_format_draft_targets.md) · [Messaging](reference_messaging_backend.md) · [Msg traps](reference_messaging_backend_shared_repo_traps.md) · [Audit log](project_central_audit_log.md)

- [Chat presets = CCS](reference_chat_permission_presets_live_in_ccs_not_rps.md) — preset list is CCS permission_group.go · what chat-core checks · backfill flags for non-system roles

## Local stack / environment traps
- [worktree add ล้มใน sandbox](reference_submodule_worktree_add_fails_in_sandbox_and_local_rps_db_via_docker.md) — .claude/.idea เขียนไม่ได้ · prune แล้ว add นอก sandbox · rps DB local = postgres:postgres
- [dev cluster + DB pod](reference_dev_cluster_access_and_readonly_db_pod.md) — dev อยู่ staging-th ns octoplus-dev/bola-dev · อ่าน log/DB (read-only pod, secretKeyRef)
- [slug เก่า×LINE login](reference_line_login_old_slug_after_rename.md) — เปลี่ยน slug แล้ว oa_not_bound · แก้ !188/!189 · destination ใน BOLA ยังฝัง slug เก่า
- [รัน QA บน local](reference_local_qa_run_facts.md) — ชื่อ DB จริง, เซสชันเว็บ, 🔴 เรทลิมิตบล็อกถาวร+วิธีล้าง
- [consent 503 บน local](reference_local_3rdparty_consent_acceptance_off.md) — 3rdparty-api ไม่มี CONSENT_ACCEPTANCE_ENABLED · ไม่ใช่ bug
- [local/stack script](reference_local_stack_branch_script.md) — scripts/local-stack.sh · local-only merge branch จาก main
- [local CRM schema ค้าง](reference_local_crm_schema_lags_repo530.md) — record หยุด 2026-09-01 · repo 530 develop ขาด 41 ไฟล์ของ main
- [internal key 401](reference_local_internal_key_exported_env_beats_dotenv.md) — key ที่ export ใน tmux ของ overmind ชนะ .env (godotenv ไม่ override)
- [TMPDIR สองที่](reference_tmpdir_differs_sandboxed_vs_unsandboxed.md) — sandbox กับ unsandboxed คนละโฟลเดอร์ · worktree ใช้ scratchpad path เต็ม
- [login local "รหัสผิด"](reference_local_kratos_wrong_password_is_unverified_email.md) — จริง ๆ คืออีเมลยังไม่ verify (hook) หรือ flow หมดอายุ · ดู log kratos ก่อน
- [Docker ครึ่งตายหลังดิสก์เต็ม](reference_docker_desktop_half_dead_after_disk_full.md) — killall -9 com.docker.backend แล้ว start · ปลุก Kafka/air/Caddy ต่อ
- [k8s env `$$`→`$`](reference_k8s_env_value_dollar_dollar_becomes_dollar.md) — SQL ผ่าน env ของ pod ใช้ `$x$` · zsh `${=s}` · rtk ตัด curl · glab mr create พังใน monorepo ใช้ glab api
- [overmind = หลาย instance](reference_overmind_is_many_instances_one_socket_per_proc.md) — ~20 instance socket ละตัว · restart ผ่าน socket ของมัน ห้าม kill pid (ทำพลาด 10-11) · `overmind start -f <Procfile> -l <proc> -s .overmind-<proc>.sock -N -D`
