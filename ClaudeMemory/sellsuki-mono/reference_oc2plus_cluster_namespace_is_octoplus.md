---
name: reference_oc2plus_cluster_namespace_is_octoplus
description: OC2Plus LINE CRM อยู่ใน namespace octoplus (staging) / octoplus-dev (dev) ไม่ใช่ sellsuki-staging-th และยิง service ตรงได้ที่ <svc>.octoplus.internal.staging-th.sellsuki.com ส่วน gateway จริงคือ crmapi.staging-th.oc2.plus
metadata:
  node_type: memory
  type: reference
  originSessionId: 7b2f9d92-0c29-4b4e-bcb2-47995813fc48
  modified: 2026-10-13T03:12:24.719Z
---

บน cluster staging-th:

- namespace คือ **`octoplus`** (staging) และ `octoplus-dev` (dev) — ไม่ใช่ `sellsuki-staging-th`
  หรือ `sellsuki` (namespace `sellsuki` มีจริงแต่เป็นของ AI chat/messaging ไม่มี pod ของ CRM)
- pod ชุด CRM: `oc2plus-line-crm-service-{3rdparty-api,backoffice-api,member-api}`,
  `-worker-{cleanup,generate-code,webhook-processor}`, `-woker-file-exporter` (สะกดแบบนี้จริง), cron อีกหลายตัว

สองโฮสต์ที่ใช้คนละงาน — อย่าสลับ:

| | URL | ใช้ทำอะไร |
|---|---|---|
| internal (ตรงเข้า service) | `http://oc2plus-line-crm-service-3rdparty-api.octoplus.internal.staging-th.sellsuki.com/v2/` | ทดสอบ scope enforcement ของตัว service โดยยัด header `X-Api-Key-Id`/`X-Company-Id`/`X-Api-Scope` เองได้ (ไม่ผ่าน Oathkeeper) — suite ของ OC-4430 ใช้เส้นนี้ |
| gateway จริง | `https://crmapi.staging-th.oc2.plus/` | เส้นที่ partner ใช้จริง Oathkeeper **ทับ** header พวกนั้นทิ้ง ยัด scope ปลอมแล้วยังได้ 401 |

ทั้งคู่ยิงจากเครื่องได้ด้วย `curl` (ใส่ชื่อโฮสต์เต็มใน allowed_domains) ทำให้รัน Robot suite
`ENV=staging` จากแล็ปท็อปได้โดยไม่ต้อง port-forward — ยกเว้น suite ที่ต้อง seed DB (ต้องมี
DB host/credential จาก env ของ CI และต่อ Postgres ตรง ซึ่ง proxy ของ sandbox เป็น HTTP อย่างเดียว)
และ suite ที่อ่าน log pod ผ่าน kubectl

เกี่ยวข้อง: [[reference_sandbox_blocks_go_cli_tls_and_ssh]]
