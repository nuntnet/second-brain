---
name: reference_manual_staging_gate_silent_drift
description: "BOLA/OC2Plus staging deploy jobs are manual gates nobody clicks — merged-to-main is NOT deployed, and the drift is invisible"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-11T04:05:24.073Z
---

ทุก repo ของ BOLA และ OC2Plus ตั้ง deploy job ของ staging เป็น **manual** ⇒ **merge เข้า main ไม่เท่ากับขึ้น staging** และไม่มีอะไรเตือนเลย

**หลักฐาน (2026-09-11):** `bola-frontend` `deploy_staging` ที่สำเร็จจริงครั้งสุดท้ายคือ **2026-07-21** (`48b3b30`) — main เดินไป **24 commits** (BOLA-309 add-existing-member / BOLA-310 suspended workspace / BOLA-313 layered gating + pending invitations / BOLA-314 Kratos password UI) โดยไม่เคยขึ้น staging เลย ~7 สัปดาห์ ทั้งที่ทุกใบ merge แล้วและการ์ดดูเหมือนเสร็จ

**ทำไมมองไม่เห็น:**
- pipeline ของ main ขึ้น **แดง** ด้วย `failed_outdated_deployment_job` (ถูก supersede ไม่ใช่เทสพัง) แล้ว `deploy_staging` กลายเป็น `skipped` — เห็นแดงแล้วเข้าใจว่า "CI พัง" ไม่ใช่ "ยังไม่ได้ deploy"
- pipeline ก่อนหน้าที่เขียวครบค้างสถานะ `manual` เฉย ๆ ไม่มีใครกด
- `deployments?environment=staging&status=success` **ไม่พอ** — job อย่าง `build_code_staging` / `prepare_staging_frontend` ก็สร้าง deployment record สีเขียว ต้องกรอง `.deployable.name == "deploy_staging"` ถึงจะเห็นของจริง

**วิธีเช็คที่เชื่อได้:**
```
glab api "projects/<enc>/deployments?environment=staging&status=success&order_by=created_at&sort=desc&per_page=40" \
  | jq -r '.[] | select(.deployable.name=="deploy_staging") | "\(.created_at)\t\(.sha[0:8])"'
```
เทียบ sha ที่ได้กับ `git rev-parse origin/main` ⇒ ต่างเมื่อไหร่คือมีของค้าง

**ข้อควรระวังเพิ่ม:**
- `bola-backend` มี **5** manual job ใน deploy stage (`deploy_staging_th_arm` + `deploy_cron_staging_arm` อีก 4 ตัว: broadcast / lon-job / rich-menu-sync / segment-refresh) — กดแค่ตัวหลัก cron จะค้างที่ image เก่า ต้องกดครบทั้ง stage
- deployment record ของ `deploy_staging_th_arm` ลงที่ environment ชื่อ **`staging`** ไม่ใช่ `staging-th` (ต่างจาก cron jobs) — กรองผิด env แล้วจะนึกว่าไม่ได้ deploy
- develop pipeline แดงด้วย `stuck_or_timeout_failure` = ไม่ได้ runner ไม่ใช่เทสพัง (ดู [[reference_stuck_ci_job_holds_resource_group]], [[reference_dead_staging_runner_tag]])

**ลำดับ deploy ที่ถูกเมื่อของข้าม service:** deploy ปลายทางที่ถูกเรียกก่อนเสมอ — OC-4514 preset ของ BOLA เรียก `GET /v1/system/company/:id/customer-app` ของ backoffice-api ถ้ากด BOLA ก่อนจะ 404 ทุกครั้ง (ยืนยันว่า route นั้นอยู่บน main แต่ไม่อยู่ใน sha ที่ staging รันอยู่ ด้วย `git merge-base --is-ancestor <commit> <deployed-sha>`)

**พิสูจน์ว่า route ใหม่ขึ้นจริงโดยไม่ต้อง login:** ยิง route นั้นแบบไม่มี auth แล้วดูรูปแบบ 404 — route ที่ลงทะเบียนแล้วตอบ JSON ของ domain (`{"error_code":"workspace_not_found"}`) ส่วน route ที่ไม่มีจริงตอบ Fiber เปล่า ๆ (`Cannot GET /v1/...`) ⇒ แยกออกได้ทันทีว่า deploy ติดหรือยัง

ใช้คู่กับ [[project_bola_deploy_topology.md]] · [[reference_env_urls]] (FE staging จริงคือ `bola-web.staging-th.bearyweb.com` ไม่ใช่ `bola.staging-th.sellsuki.com`)

---

**Sweep 43 repo (2026-09-11):** ตรงกับ main แล้ว 20 · **ค้าง 14** · ไม่มี staging deploy job เลย 8 (space-go, space-storefront, catalog-service, sellsuki-chat-core, sellsuki-ai-agent, ai-platform-kit-go, pis-admin, shipmunk-frontend) ⇒ การค้างเป็น **ค่าเริ่มต้นของ workspace นี้ ไม่ใช่เคสพิเศษ**

ตัวที่ค้างหนักและ **อาจตั้งใจ ห้ามดันโดยไม่ถาม**: `central-configuration-system` 53 commits (ทุก service อ่าน config จากมัน) · `rag-core` 56 · `ai-chat-admin-frontend` 70 — สองตัวหลังอยู่ใต้ AI platform ที่มี `CI_JOB_ENABLE` gating (ดู [[project_ai_platform_deploy_gating]]) · `paper-backend` ค้างตั้งแต่ เม.ย. 2024

**แดงเพราะ runner ไม่ใช่โค้ด — retry แล้วหาย:** `stuck_or_timeout_failure` กับ `runner_system_failure` เจอพร้อมกัน 3 repo ในวันเดียว (messaging-backend, ai-chat-admin-frontend, sellsuki-invitation) ⇒ เห็น pipeline แดงบน main ให้อ่าน `failure_reason` ก่อนสรุปว่าเทสพัง

**🔴 `quota-management-backend` → `deploy_renew_cronjob_staging_th_arm` ไม่เคยสำเร็จเลย** (fail ทั้ง 8 ก.ย., 10 ก.ย. และรอบ 11 ก.ย.): helm บ่น `unknown object type "nil" in Ingress.metadata.annotations.external-dns.alpha.kubernetes.io/hostname` เพราะ CI ส่ง `--set internalIngress.enabled=${INTERNAL_INGRESS_ENABLED}` จาก SRE template โดย `INTERNAL_INGRESS_DOMAIN` ว่างสำหรับ release ชื่อ `${CI_PROJECT_NAME}-renew-cronjob` — **CronJob ไม่ควรมี Ingress** และ `--set` ชนะ `-f` เสมอ จึงแก้ที่ values ไม่ได้ ต้องใส่ `INTERNAL_INGRESS_ENABLED: "false"` ใน `variables:` ของ job นั้น (`.gitlab-ci.yml:126`) · มี `--atomic` ⇒ install ที่ fail ถูก rollback ไม่มีของค้างครึ่งทาง · **ตัว API หลักขึ้นปกติ** ค้างแค่ cronjob
