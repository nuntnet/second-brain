---
name: reference_staging_chat_core_replay_recipe_in_pod_wget
description: "วิธี replay ข้อความลูกค้าเข้า chat-core บน staging โดยไม่แตะ token: kubectl exec เข้า pod chat-core แล้วใช้ busybox wget POST /internal/workspaces/{ws}/messages ด้วย $SERVICE_TOKEN ใน pod · ถ้าต้องการ bubble ติดกันต้องยิงทั้งชุดใน exec เดียว (sleep 1 ใน pod) เพราะ kubectl exec ผ่าน teleport ช้า/ค้างได้เป็นนาที · อ่านผลจาก psql ใน pod postgresql-pg18-0 (ns datastore, -U postgres, db chat_core): messages / inbound_ai_work / decision_traces / fallback_cases · metric chat_core_reply_errors_total{reason=\"turn_superseded\"} ที่ /metrics port 80"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-22T06:37:01.416Z
---

ใช้จริง 2026-10-22 (AI-298 tier B ข้อ B/C):

```bash
# ส่ง 3 bubble ห่าง 1 วิ จากใน pod เดียว (token ไม่ออกมานอก cluster)
kubectl -n sellsuki exec deploy/sellsuki-chat-core -- sh -c "for d in '<json1>' '<json2>' '<json3>'; do wget -qO- --server-response --header='Content-Type: application/json' --header=\"X-Service-Token: \$SERVICE_TOKEN\" --post-data=\"\$d\" http://127.0.0.1:80/internal/workspaces/<ws>/messages 2>&1 | grep -m1 HTTP/; sleep 1; done"
# json: {"external_message_id":"m_<unique>","channel_identity_ref":"facebook:<synthetic>","channel_conversation_id":"<uuid>","channel":"facebook","body":"..."}
# อ่านผล
kubectl -n datastore exec postgresql-pg18-0 -c postgresql -- psql -U postgres -d chat_core -tA -F' | ' -c "select m.seq, m.sender_type, m.tier from messages m join chat_sessions s on s.id=m.chat_session_id where s.channel_identity_ref='facebook:<synthetic>' order by m.seq;"
```

- ตาราง durable inbound ชื่อ `inbound_ai_work` (state pending/processing/completed/suppressed, คอลัมน์ customer_message_id) ไม่ใช่ `inbound_work`; outbox คือ `inbound_ai_answer_outbox`
- ผลที่ถูกของ !151+!152: bubble 1–2 `suppressed`, bubble 3 `completed`, AI reply 1 แถว, decision_traces 1 แถว kb_chunks > 0, metric `chat_core_reply_errors_total{hop="session_resolve",reason="turn_superseded"}` (pre-check) และ `{hop="llm_call",…}` (post-check)
- กับดัก: ถ้า bubble แรกมาเดี่ยว ๆ แล้ว agent ตอบ confidence < 0.6 → `fallback_cases low_confidence` + human_mode → bubble ถัดไปถูก suppressed ด้วย "คนดูแล" ไม่ใช่ supersede (attempts 0) — แยกให้ออกจากผล supersede (attempts 1)
- ห้องสังเคราะห์โผล่ใน inbox ของ workspace pilot จริง (บางห้องเป็น "ต้องการคน") ต้องปิด/บอก PO
- agent ไม่ log error ของ provider; span ไป OTel collector ซึ่งอาจปฏิเสธ (`high memory usage`) → หาเหตุด้วย probe `/internal/v1/workspaces/{ws}/reply` (ต้องส่ง config.model) และ `/internal/v1/llm` (nc ใน pod เพื่อเห็น body ของ 4xx; busybox wget โชว์ body ตอน error ไม่ได้) · ดู breaker ที่ `ai_agent_circuit_breaker_state`
- เกี่ยวข้อง: [[reference_chat_core_local_inbound_forward_for_live_tests]], [[reference_ai_agent_staging_vault_token_expiry_crash_on_restart]], [[reference_staging_ai_chat_console_drive_recipe]]
