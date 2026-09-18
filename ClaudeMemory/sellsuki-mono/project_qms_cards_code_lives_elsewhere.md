---
name: project_qms_cards_code_lives_elsewhere
description: "QMS-UI การ์ด 21 ใบสถานะ code review มีโค้ดจริงครบ แต่กระจายใน 3 MR ที่นิ่งตั้งแต่ 2026-07-20 และการ์ด 12 ใบระบุรีโปผิด"
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
