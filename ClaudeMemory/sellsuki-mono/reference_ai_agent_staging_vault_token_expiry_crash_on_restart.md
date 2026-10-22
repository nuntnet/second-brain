---
name: reference_ai_agent_staging_vault_token_expiry_crash_on_restart
description: "2026-10-22 — agent staging ล่มหลัง deploy เพราะ Vault ปฏิเสธ VAULT_TOKEN (403) ตอน boot (AI-88 AC3 ไม่มี fallback) · pod เก่ารอดเพราะถือ credential ในหน่วยความจำ → อาการโผล่ตอน restart ครั้งแรกหลัง token เสีย · helm timeout + rollback ก็ crash เหมือนกัน · secret มาจาก ExternalSecret KV kubernetes-secret/sellsuki/sellsuki-ai-agent-common-secret (ESO sync ปกติ, ค่า VAULT_TOKEN ข้างในเสีย) · แก้ได้เฉพาะคนที่มีสิทธิ์ Vault; classifier ห้าม decode ค่า secret แม้บางส่วน"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-22T03:23:30.219Z
---

เห็น 2026-10-22 (AI-81 / AI-298 /land): `deploy_staging_th_amd` ของ sellsuki-ai-agent ล้มด้วย `helm … timed out waiting for the condition` ทั้ง upgrade และ rollback; pod `sellsuki-ai-agent-0` CrashLoopBackOff แม้ image เก่า (e5e7eab5) — log: `llm credential: credentials: initial load from vault: credential_repository: vault rejected this service's token (403)`.

- main.go refuse to boot ใน staging/prod ถ้า Vault โหลดไม่ได้ (AI-88 AC3, ไม่มี `LLM_API_KEY` fallback โดยตั้งใจ) → token หมดอายุ/ถูก revoke จะไม่ล้มทันที แต่ล้ม "ครั้งถัดไปที่ pod restart" ซึ่งมักเป็นตอน deploy → ดูเหมือน deploy พัง
- ตรวจเร็วจากนอก cluster: `GET https://chat-core.staging-th.sellsuki.com/v1/companies/<id>/quota/status` (fetch จาก pane ของ console ที่ล็อกอินแล้ว) → 503 `quota_unavailable` = agent ไม่ตอบ
- secret `sellsuki-ai-agent-common-secret` เป็น ExternalSecret `dataFrom.extract key kubernetes-secret/sellsuki/sellsuki-ai-agent-common-secret` บน ClusterSecretStore `vault-backend` refresh 300s — SecretSynced True ไม่ได้แปลว่า token ข้างในใช้ได้
- agent เป็น StatefulSet 1 replica (PVC spool) → rolling update ลบ pod เก่าก่อน = ไม่มี pod ดีเหลือระหว่าง deploy; helm rollback คืน image เก่าแล้วก็ crash ต่อ ต้อง retry job deploy ใหม่หลัง token แก้
- auto-mode classifier ปฏิเสธการ decode secret แม้พิมพ์แค่ความยาว/prefix ([Credential Materialization]) — ดูได้แค่ชื่อ key, metadata, ExternalSecret spec

**How to apply:** deploy ของ agent ล้มด้วย helm timeout → ดู `kubectl -n sellsuki logs sellsuki-ai-agent-0 --previous` ก่อน retry; ถ้าเป็น vault 403 ให้หยุด retry แล้วส่งให้คนที่มีสิทธิ์ Vault ออก token ใหม่เขียนทับ field `VAULT_TOKEN` ใน KV ข้างบน (ESO sync ≤5 นาที) แล้วค่อย retry deploy · ถ้า secret สร้าง/หมุนครั้งล่าสุดราว 30 วันก่อน ให้สงสัย TTL ก่อน · เสนอ SRE ใช้ periodic token หรือ Vault k8s auth · เกี่ยวข้อง: [[reference_night_window_is_staging_th_only_production_runners_run]], [[feedback_morning_retries_in_waves_runner_cluster_is_12_nodes]]

**แก้จริง 2026-10-22 10:47 (PO รันเอง):** `export VAULT_ADDR=https://vault.internal.staging-th.sellsuki.com` · token ของ PO ออก token ด้วย `-policy=` ไม่ได้ ("child policies must be subset of parent") → ใช้ token role: `vault token create -role=ai-agent-sellsuki -field=token` (role: policy `ai-agent-credential-sellsuki` = kubernetes-secret/{data,metadata,delete}/sellsuki/ai-agent-llm-credential, period 720h, orphan) → `vault kv patch kubernetes-secret/sellsuki/sellsuki-ai-agent-common-secret VAULT_TOKEN=…` (patch ไม่ใช่ put) → `kubectl -n sellsuki annotate externalsecret sellsuki-ai-agent-common-secret force-sync=$(date +%s) --overwrite` → ลบ pod-0 → boot ผ่าน · role อีกตัว `messaging-chat-staging` ใช้วิธีเดียวกัน · **จะตายอีกทุก 30 วันถ้าไม่มีใคร renew** — ต้องมี cron renew หรือ k8s auth

**ปัญหาซ้อนที่เจอวันเดียวกัน (2026-10-22) หลัง token ใช้ได้:** (1) OpenRouter api_key ใน KV `kubernetes-secret/sellsuki/ai-agent-llm-credential` ใช้ไม่ได้ (alias ****9a7c ไม่ตรง key ใดในบัญชี) → agent ล้มทุก call, breaker gemini เปิด; agent นับทุก error เข้า breaker รวม 401 → PO `vault kv patch … api_key=<ใหม่>` แล้ว agent หยิบเองใน 30 วิ (`llm credential rotated v1 -> v2`) ไม่ต้อง restart · ตรวจบัญชี/credits/keys ได้ด้วย `OPENROUTER_MANAGEMENT_KEY` ใน pod agent (`/api/v1/credits`, `/api/v1/keys`) · `LLM_API_KEY` ของ chat-core ก็ 401 (ใช้แค่ Tier-1 embedding) (2) `sellsuki-rag-core-secret` version 20 ถูกเขียนทับแบบหลุด field (`GATEWAY_JWT_PUBLIC_KEY_PEM`, `POSTGRES_DSN`) → pod rag-core-python ที่ถูกสร้างใหม่ตอน shutdown ติด CreateContainerConfigError 16 ชม. → คืนจาก v19 ด้วย `vault kv patch` ไล่ทุก field ที่หาย (เทียบ `vault kv get -version=N -format=json` กับปัจจุบัน) แล้ว force-sync ESO + ลบ pod · ก่อนประกาศ "ครบ" เทียบกับ secretKeyRef ของทุก deploy/cronjob ใน ns (`GOOGLE_SHEETS_API_TOKEN` ไม่เคยมีเลย = ช่องว่างเดิมของ sheet-sync cron)

