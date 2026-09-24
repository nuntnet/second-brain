---
name: reference_ccs2_dev_is_a_stale_develop_build
description: ccs.dev.sellsuki.com (provider-management-frontend, CCS2) = build จาก `develop` ที่ deploy ครั้งสุดท้าย 2026-03-16 · ของจริงอยู่บน `main` ซึ่ง pipeline มีแต่ deploy_staging แบบ manual · ฟีเจอร์ที่ merge main (เช่น QMS UI !7) จึงไม่อยู่ทั้ง dev และ staging จนกว่าจะมีคนกด
metadata:
  type: reference
---

พบ 2026-09-24 ตอนตรวจ OC-4570 AC-02 ("หน้า Quota บน dev"):

- `frontend/sellsuki-provider-management-frontend` `.gitlab-ci.yml`: develop → `ccs.dev.sellsuki.com`, main → `ccs.staging.sellsuki.com`, tag → `ccs.sellsuki.com`
- `develop` ค้างที่ 1bc922d (2026-03-10) ตามหลัง `main` 118 commit · deploy_development ครั้งสุดท้าย job 186434 2026-03-16
- `main` pipeline (เช่น 59382 ของ QMS UI 551edfd) มี `deploy_staging` เป็น **manual** และไม่มีใครกด — ฟีเจอร์ merge แล้ว 6 วันแต่ไม่อยู่ที่ไหนเลย
- วิธีตรวจว่า build ไหนอยู่บน host: `curl` index → หา `/assets/index-*.js` → grep สตริง API path (`qms/quotas`) ในบันเดิล — ชื่อคอมโพเนนต์ถูก minify แต่ URL string ไม่หาย

**2026-09-24 13:38 กด `deploy_staging` (job 249532) แล้ว** → `ccs.staging.sellsuki.com` มี QMS UI · แต่ **management-backend บน staging (ns `patona`) = main ที่ไม่เคยมี QMS proxy** — `/v1/qms/quotas` 404 · โค้ด QMS อยู่ develop เท่านั้น (นำ main 52 commit) ⇒ **dev มี API ไม่มี UI · staging มี UI ไม่มี API** · management-backend สายจริงคือ develop (ดู [[reference_real_mainline_per_repo]]) และห้ามเปิด develop→main เอง ([[feedback_no_develop_to_main_promotion_mrs]])

**How to apply:** การ์ดที่เขียน "ตรวจบน dev" สำหรับ CCS2 ต้องถามก่อนว่า deploy ทางไหน — dev ของ repo นี้ไม่ใช่ที่ที่โค้ดใหม่ไป · ดู [[reference_real_mainline_per_repo]] · [[project_qms_ui]]
