---
name: reference_line_login_old_slug_after_rename
description: เปลี่ยน slug แอปสมาชิก (OC-4560) แล้วทางเข้า LINE (rich menu/LIFF ที่ฝัง slug เก่า) ล้ม oa_not_bound — LINE login/link ต้อง resolve ผ่าน slug history; destination ใน BOLA ไม่ถูกอัปเดตเอง
metadata:
  type: reference
---

**เหตุการณ์ 2026-10-02 dev:** บริษัท fa72f8bf… เปลี่ยน slug `daudmsv10pjbo30tkfbg`→`autotestmaha` เวลา 09:46 UTC · `oc2plus_bola_bindings` เก็บแค่ slug ปัจจุบัน, slug เก่าอยู่ `member_app_slug_history` · `ResolveCompanyBySlug`/ธีมดู history แต่ `LineIDTokenLogin`/`LinkLineIdentity` (OC-4523) เรียก `GetBySlug` ตรง ๆ → 403 `oa_not_bound` ทุกครั้งที่เปิดจากเมนู LINE เก่า (ซ่อนเหตุผลจริงเพราะรวมหลายกรณีเป็นรหัสเดียว) → คนใหม่ถูกส่งไปกรอกเบอร์ สมัครแล้วไม่ถูกผูก LINE → หน้า Contacts ใน BOLA เห็น 2 รายการ (Member ไม่มี LINE + Follower ไม่มีเบอร์)

**แก้แล้ว:** member-api !188 (`bindingForLineSlug`: history บอกแค่บริษัท แล้วอ่าน binding ตามบริษัท การตรวจที่เหลือเหมือนเดิม) + !189 (log `cause` 7 แบบ: slug_not_found, slug_lookup_failed, company_not_open, open_check_failed, no_line_oa_in_request, oa_not_in_workspace, slug_of_another_company — client/audit เห็นรหัสเดียว) · เทียบ 10 endpoint `{slug_company}` slug เก่า = ใหม่ (auth/line จาก 403 → 401)

**ยังเป็นความเสี่ยง:** `liff_shell_destinations` ใน BOLA ยังฝัง slug เก่า (เปลี่ยน slug ไม่อัปเดตเมนู/destination) · การ์ด OC-4560 และแผน QA (OC-4663/4664) ตรวจ "ลิงก์เก่า" ด้วย `/availability` เท่านั้น ไม่ครอบทางเข้า LINE

**How to apply:** เมื่อเห็น `oa_not_bound` ให้ดู `cause` ใน log ก่อนเดา · ทุก endpoint ใหม่ที่รับ slug ต้องผ่านตัว resolve เดียวกับที่ดู history
