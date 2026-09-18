---
name: reference_real_mainline_per_repo
description: "default_branch ของ GitLab เป็น main ทั้ง 55 repo แต่สายจริงต่างกันรายรีโป — ตารางว่าอันไหน develop อันไหน main และหกตัวที่ develop เป็นป้ายเปล่า"
metadata:
  node_type: memory
  type: reference
  modified: 2026-09-18T15:00:00.000Z
---

กวาดทุก submodule 2026-09-18 · **`default_branch` = `main` ทั้ง 55 repo ไม่มีข้อยกเว้น
และค่านั้นเชื่อไม่ได้** — central-configuration-system มีงานบน develop 85 commit
แต่ยังรายงาน default เป็น main

**เช็คก่อนเปิด MR ในรีโปที่ไม่คุ้น:**
```bash
git rev-list --left-right --count origin/main...origin/develop   # ซ้าย=main only  ขวา=develop only
```

**develop คือสายจริง (ขวาสูง):** central-configuration-system 85 · management-backend 48 ·
sellsuki-service-consent 33 · rps 31 · sellsuki-system-management-frontend 29 ·
file-service / address-backend 18 · catalog-service / customer-book-backend-v2 /
oc2plus-linecrm-frontend-backoffice 17 · oc2plus-line-crm-service-backoffice-api 16 ·
sellsuki-inventory-management 15 · order-management-backend / kratos-ui-go /
sellercenter-frontend / sellsuki-central-control-backend 11-13 · catalog-frontend 3

**🔴 develop เป็นป้ายเปล่า (`develop+0`) — เลือกผิดแล้ว MR จะเงียบหาย:**
`sellsuki-invitation` (main+35) · `sellsuki-provider-management-frontend` (main+18) ·
`paper-backend` (main+13) · `bola-backend` (3) · `bola-frontend` (2) ·
`oc2plus-line-crm-service-3rdparty-api` (1)

**ไม่มี develop เลย (ปลอดภัย):** space-go · space-storefront · rag-core · sellsuki-chat-core ·
sellsuki-ai-agent · ai-platform-kit-go · ai-chat-admin-frontend · shipmunk-frontend ·
pis-admin · testing/* เกือบทั้งหมด

**ยังไม่ชี้ขาด (0/0):** pis-api · pis-frontend · pis-cronjob-file-cleaning-up

## เคสที่ทำให้รู้ — OC-4482/QMS MR !7

`sellsuki-provider-management-frontend` !7 ตั้ง target เป็น `develop` ทั้งที่
**MR ทุกใบในประวัติรีโปตั้งแต่ปี 2024 (ใบ 1-6, 8-10) ชี้ `main` หมด** ·
`develop` ชี้ commit `1bc922d` ที่ข้อความคือ "Merge ... into 'main'" — คือ commit ที่เกิดบน main
ไม่ใช่สายพัฒนา ไม่มี commit ของตัวเองเลย แตะล่าสุด 2026-03-10

ผลสองชั้นจากรากเดียว: MR ไม่มีใครเห็นเพราะชี้ที่ไม่มีใครดู **และ** CI ไม่รันเลย 6 เดือน
เพราะ branch แตกก่อน `4d5f4a0` "fix(ci): pin merge_request jobs to staging-th runner tags"
จึงขอ runner tag `staging` ที่ไม่มี runner ตัวไหนบนออร์กมีแล้ว (ทุกตัวเป็น `staging-th`)
ดู [[reference_dead_staging_runner_tag]]

เดาที่มีหลักฐาน: ความเคยชินจาก OC2Plus ที่ develop **คือ** mainline จริง
([[feedback_oc2plus_merge_to_develop]]) แต่รีโปนี้อยู่ใต้ `sellsuki/frontend`

ดู [[project_monorepo_mainline_is_not_main]] · [[feedback_no_develop_to_main_promotion_mrs]]
