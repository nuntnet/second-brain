---
name: reference_messaging_backend_staging_vault_token_403_makes_fb_webhook_500
description: "2026-10-24 staging: 'ทักเพจแล้วไม่มี AI ตอบ' = messaging-backend ตอบ POST /webhook/fb/<app> 500 ใน ~4ms ไม่มี log เพราะ VAULT_TOKEN (secret sellsuki-messaging-common-secret) หมดอายุ → Vault 403 → bare error → unexpected_error · chat-core ไม่เห็นข้อความเลย · พิสูจน์ด้วย wget lookup-self ในพอด · ต้องคนมีสิทธิ์ Vault ออก token ใหม่"
metadata:
  type: reference
---

อาการ: ผู้ใช้ทักเพจ staging แล้วไม่มีคำตอบ ทั้งที่ chat-core ปกติ (`inbound_ai_work` ไม่มีแถวใหม่เลย = ข้อความไม่ถึง chat-core).

วิธีไล่ (ใช้ได้อีก):
1. `inbound_ai_work` ล่าสุดเมื่อไร (ns datastore, db chat_core)
2. `kubectl -n sellsuki logs deploy/sellsuki-messaging-backend` ดู `request_log` ของ `/webhook/fb/:appId` — status 500 + duration ~4ms = ล้มก่อนทำอะไร
3. พิสูจน์ Vault ในพอด: `kubectl -n sellsuki exec deploy/sellsuki-messaging-backend -- sh -c 'wget -qO- --server-response --header="X-Vault-Token: $VAULT_TOKEN" "$VAULT_ADDR/v1/auth/token/lookup-self"'` → 403 = token ตาย (token ไม่ออกนอกพอด)

ทำไมเงียบ: `vault_http.go` คืน error เปล่า ๆ เมื่อสถานะไม่ใช่ 2xx/404 → `fb_webhook/errors.go` แมปเป็น 500 `unexpected_error` ไม่ log (เฉพาะ 404 ถึงจะเป็น `ErrFBAppSecretUnresolvable` 503 + log). Meta retry แล้วอาจปิด subscription ถ้าปล่อยนาน.

ข้อดีที่ได้มา: ยืนยันว่า Meta → staging webhook ถึงจริง (AC9 routing ที่เคยไม่ชัวร์). แก้จริง = ผู้มีสิทธิ์ Vault ออก token ใหม่ใส่ secret `sellsuki-messaging-common-secret` แล้ว restart messaging-backend (+ service-messaging ใช้ secret เดียวกัน). Follow-up โค้ด: ห่อ Vault 401/403 เป็น `ErrFBAppSecretUnresolvable`.

เกี่ยวข้อง: [[reference_ai_agent_staging_vault_token_expiry_crash_on_restart]], [[reference_staging_chat_core_replay_recipe_in_pod_wget]], [[reference_sellsuki_new_meta_app_state]]
