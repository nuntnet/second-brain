---
name: project_qms_cards_code_lives_elsewhere
description: "QMS-UI 21 ใบ: MR !194 (management-backend→develop) และ !7 (provider-management-frontend→main) MERGE แล้ว 2026-09-18 — แต่ QMS_API_KEY/SECRET ไม่ถูกตั้งทุก env ⇒ backend ตอบ dummy เงียบ ๆ (OC-4570) · FE ยังไม่เคย deploy (deploy_staging manual) · mainline ของ FE คือ main"
metadata:
  node_type: memory
  type: project
  modified: 2026-09-18T14:30:00.000Z
---

ตรวจ 2026-09-18 จากโค้ดจริง ไม่ใช่สถานะ Jira

**ทั้ง 21 ใบที่อยู่ "code review" มีโค้ดครบทุกใบ** — ไม่ใช่งานที่ยังไม่ทำ
ค้างเพราะไม่มีใคร review/merge มาสองเดือน ทั้งสาม MR commit ล่าสุด **2026-07-20**

| MR | ครอบการ์ด | สถานะ |
|---|---|---|
| `management-backend` !194 | OC-4260, 4279, 4280, 4374, 4375, 4376 | 🔴 conflict |
| `sellsuki-provider-management-frontend` !7 (branch `feat/qms-admin-ui-oc4257`) | OC-4259, 4262, 4264, 4265, 4266, 4268, 4269, 4270, 4271, 4272, 4273, 4274, 4281, 4372 | 🔴 pipeline failed |
| `quota-management-api-automate-testing` !1 | OC-4275 | opened |

🔴 **กับดักที่ทำให้ผมสรุปผิดสองรอบติด:**

1. **การ์ด 12 ใบระบุรีโปผิด** — เขียนว่า `sellsuki-system-management-frontend`
   แต่โค้ดจริงอยู่ใน **`sellsuki-provider-management-frontend`** ·
   รีโป system-management-frontend **ไม่มี branch QMS เลยสักอัน** ใครเชื่อการ์ด
   แล้วไปหาที่นั่นจะสรุปว่า "ไม่มีใครทำ"
2. **หาไฟล์ตามชื่อที่เดาเอง** — OC-4274 "Transaction Detail and event log" ไม่มี
   `TransactionDetailPage.svelte` เพราะ AC ของมันเองบอกว่า events มาแบบ inline
   ตาราง events อยู่ใน `TransactionListPage.svelte` · OC-4275 ก็เช่นกัน การ์ด
   ลิงก์ MR ไว้ในคำอธิบายตั้งแต่แรก แต่ผมไปดู branch ใน management-backend แทน

**หน้า UI ที่มีจริงใน branch `feat/qms-admin-ui-oc4257`** (17 ไฟล์ใต้ `src/pages/qms/`):
QmsHome · QuotaList/Detail/Create/Form/Upgrade/UsageHeader · PlanList/Create/Edit/Detail ·
AssignPlanList/Create/Detail · CompanyPicker · CheckSummarize · TransactionList
\+ API client ที่ `src/services/qms-rest.ts` / `qms-model.ts`

**สิ่งที่ต้องตัดสิน:** ปลุก 3 MR หรือปิดทิ้ง · ถ้าปลุกคือแก้ conflict กับ pipeline
ไม่ใช่เขียนใหม่ · ตัดออกจาก [[project_oc_deploy_prd_sprint131]] รอบนี้เพราะยังไม่ merge

ดู [[feedback_verify_absence_claims]] · [[feedback_search_before_declaring_gap]] ·
[[project_qms_ui]] · [[reference_ai_board_in_review_means_merged_unverified]]


## Update 2026-09-22 — ทั้งสอง MR merge แล้ว ข้อความข้างบนเก่าไป 4 วัน

ตรวจจาก `glab api` วันนี้: **!194 merged → develop 2026-09-18 07:49** (แล้ว fix/qms-env-development
ตามมาเพิ่ม QMS_GRPC_SERVER/PIS_API_BASE_URL ให้ pipeline #59389 deploy dev สำเร็จ) ·
**!7 merged → `main` 2026-09-18 09:08** — `main` คือ default branch จริงของ
`sellsuki-provider-management-frontend` (develop ที่นั่นเป็น branch ค้างเก่า) อย่าไปแฟล็กว่า target ผิด ·
!1 (quota testing) ยังเปิด Draft pipeline แดง เพราะ path ในเทสต์ (`/qms/quotas/create`) ไม่ตรง route จริง
(`/v1/qms/quotas`) = OC-4569

🔴 **blocker ตัวจริงที่ Jira มองไม่เห็น:** `cmd/generics_server/helper.go:272` ถ้า `QMS_API_KEY==""`
จะใช้ `NewDummy()` ทั้ง quota+assign-plan repo และ log แค่ Info — ค่านี้ **ไม่มีใน
values-development/staging/production.yml เลย** ⇒ ทุก Quota/AssignPlan/Transaction/CheckSummarize
บน dev ตอบ dummy อยู่ (= OC-4570) · Plan CRUD (OC-4280/4281) กับ PIS product search (OC-4279)
ไม่ผ่าน gate นี้ ใช้จริงได้เลย

FE ไม่มี dev deploy — pipeline ของ main มี `deploy_staging: manual` ไม่เคยกด ⇒ ยังไม่มี URL ให้ QA

ต้นเหตุ "การ์ดระบุรีโปผิด": `sellsuki-provider-management-frontend/package.json` ตั้ง
`"name": "sellsuki-system-management-frontend"` — ใครอ่าน package.json แทน remote จะสรุปผิด ·
วันนี้เช็ค body การ์ด 18 ใบใน sprint 130 ไม่มีใบไหนใช้ชื่อผิดแล้ว เหลือแค่ **title ของ OC-4259**
