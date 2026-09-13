---
name: project_oc4363_code_redemption_state
description: "OC-4363 กรอกรหัสรับแต้ม — สถานะ 2026-09-13 หลังแก้ engine/BFF/FE ให้แยก \"ใช้ไปแล้ว\" (409) และ \"หมดอายุ\" (410); ข้อจำกัดที่เหลือคือ inquiry ของรหัสที่แคมเปญจบแล้วยังอ่านเป็น invalid"
metadata: 
  node_type: memory
  type: project
  originSessionId: fa22ebb1-f715-4667-85a1-61ebbcc816ab
  modified: 2026-09-13T04:03:55.217Z
---

Ready to test (DEV) ตั้งแต่ 2026-09-13 · commits: 3rdparty-api `715a01d` (branch `codex/oc-4344-web-member-identity`, MR !242),
member-api `4eafe88` (branch `codex/oc-4344-customer-app-integration` — MR !115 merge ไปแล้ว ต้องมี MR ใหม่จาก branch เดิม), member FE `0ac135f` (MR !74 Draft ของ Codex)

**สิ่งที่การ์ดห้ามแต่ PO เคาะให้ทำ:** การ์ดเขียน "ห้ามแก้ engine" แต่ตาราง 409/410 ของการ์ดเกิดไม่ได้เลยถ้าไม่แตะ —
engine `getCodeTemplateByCode` ค้นเฉพาะ `status='active'` จึงตอบ `invalid_code` เหมือนกันหมด · PO สั่ง "ทำ OC-4363 ก่อน"
→ เพิ่ม `CodeRepository.GetCodeByCodeAnyStatus` (scope company เสมอ) + `ErrCodeAlreadyUsed` = `400 code_already_used`
วางระหว่าง lookup active แบบ single กับ `verifyChecksum` · deactivated/บริษัทอื่น → ยัง invalid (Rule 3)

**ข้อจำกัดที่เหลือของ AC-04 ("หมดอายุ"):** engine CodeInquiry ใช้ `FindActiveCampaignByCodeCondition` — รหัสที่แคมเปญจบแล้ว
ได้ **200 ไม่มีแคมเปญ** ไม่ใช่ `campaign_expired` → BFF `InquireMyCode` แปลง list ว่างเป็น `ErrInvalidCode` → สมาชิกเห็น
"รหัสไม่ถูกต้อง" ตั้งแต่ขั้น inquiry · `campaign_expired` โผล่เฉพาะตอน **confirm** (CodeRedemption ตรวจวัน) → BFF map
เป็น 410 CODE_EXPIRED เฉพาะเส้น code (`thirdparty_repository/code.go` เช็คก่อน `isRedemptionUnavailableCode`;
เส้น campaign/point redeem ใน `http.go:182` ยัง fold) · จะให้ inquiry บอก "หมดอายุ" ต้อง query แคมเปญที่ผูก template
แบบไม่กรอง active + ตัดสินว่า template ที่ผูกหลายแคมเปญนับวันจบตัวไหน = การ์ดแยก ยังไม่มี

**Why:** ถ้าเทสบน DEV แล้วเห็น "รหัสไม่ถูกต้อง" ตอน inquiry รหัสหมดอายุ อย่าไปไล่หา bug — เป็นข้อจำกัดที่รู้และจดไว้ใน comment การ์ดแล้ว
**How to apply:** wire codes ฝั่ง BFF อ่านจาก `error_code` (ไม่ใช่ `code`); FE `CodeError` มี `alreadyUsed | expired`; copy ตามตาราง i18n ของการ์ดตรงตัว

เชื่อม [[project_oc2plus_customer_bff_reads_direct_not_proxy]] [[feedback_user_runs_codex_in_parallel]] [[reference_oc2plus_member_app_local_qa_session]]
