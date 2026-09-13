---
name: project_oc4523_line_login_cards
description: OC-4523 (verified-LINE login ทุกหน้าของ customer app) + BOLA-328 (shell แนบ id_token) — เขียนการ์ดครบ 2026-09-13 พร้อมข้อตัดสินใจ fragment/React/403 ที่ยังรอ PO ยืนยัน
metadata: 
  node_type: memory
  type: project
  originSessionId: d0b39379-feaa-4fd5-b115-18617c202159
  modified: 2026-09-13T14:44:10.180Z
---

**OC-4523 rewrite + BOLA-328 สร้างใหม่ (2026-09-13)** — คู่การ์ด "กดจาก rich menu แล้วเข้าได้เลย ไม่ต้อง OTP ซ้ำ"

**BOLA-328** `[BOLA][LIFF Shell] shell แนบ LIFF id_token ใน handoff + oa-info คืน LIFF channel id` — Story, To Do, High, Blocks → OC-4523. Size S. ทำ 2 จุด: shell เรียก `liff.getIDToken()` แนบใน fragment, และ `GET /v1/public/liff/oa-info` คืน `liff_id` + `liff_channel_id` เพิ่มจาก `name` เดิม

**สถานะโค้ดที่ตรวจแล้ว (2026-09-13) — ยังไม่มีอะไรรองรับทั้งสองฝั่ง:**
- shell ส่งแค่ 5 ค่า `line_user_id, line_oa_id, display_name, picture_url, follow_status` (`LiffShellPage.tsx:197-201`) ไม่มี `getIDToken` ในทุก remote branch
- `oa-info` คืนแค่ `name` (`route/v1/liff/model.go:20-22`)
- member-api ไม่มี `auth/line` ในทุก branch; line client มีแค่ `VerifyAccessToken` (`line_repository/rest.go:90`)
- `BolaBinding` ใน member-api มีแค่ company_id / line_oa_id / binding_status — ไม่มี LIFF id
- preset OC-4514 merge เข้า develop+main แล้ว (`oc2plus_standard_menu.go`) สร้าง 3 destination ครบ ⇒ **ไม่ต้องทำ preset ใหม่**

**id_token ≠ destination:** destination (ตาราง `liff_shell_destination`) ตอบ "ไปไหน", id_token ตอบ "ใครมา" การ์ดนี้ไม่แตะตาราง destination และไม่แตะ preset เลย flow ใหม่ในอนาคตแค่เพิ่มแถว destination แล้ว token ติดไปเอง

**3 ข้อตัดสินใจในการ์ด:**
1. ✅ **ทำบน React ไม่ใช่ Vue — PO เคาะแล้ว 2026-09-13** ("ทำบน React ได้เลย ตามที่เสนอ") **แทนที่ข้อเคาะ 2026-09-10 ที่ว่าให้ทำ Vue ก่อน** เหตุผล: ตอนนั้น OC-4501 ยังไม่เสร็จ วันนี้ Ready to test (DEV) แล้ว ⇒ จุดแก้ FE = ชั้น session ของ React shell + หน้า login React ไม่แตะ `LoginView.vue` (694 บรรทัด) · ผลพลอยได้: ข้อ "OC-4501 ต้องครอบ path LINE login ด้วย" ในคอมเมนต์เก่าตกไป เพราะ OC-4501 ส่งงานไปก่อนแล้ว
2. **id_token ใน URL fragment ไม่ใช่ query** — กัน token โผล่ใน access log / Referer (การ์ดเดิมเขียนเป็น query) ยังไม่มีใครทักท้วง
3. **เพิ่ม 403 `oa_not_bound`** แยกจาก 404 `member_not_linked` — คนละเงื่อนไข ยังไม่มีใครทักท้วง

**ซี้ที่พลาดง่าย: FE เช็ค session 2 ที่ ไม่ใช่ที่เดียว** — `AuthGuard.tsx:58` เด้ง anonymous ทันที แต่ point-claims ทั้ง 3 หน้า **ข้าม guard** แล้วเช็ค 401 เอง (`publicRoutes.ts`, ตัดสินใจใน OC-4503 เพราะ E2E TS-013 ล็อกพฤติกรรมไว้) ⇒ แก้แค่ AuthGuard จะหลุด 3 หน้าที่สำคัญที่สุด การ์ดจึงมี Rule 6 + AC-5 บังคับ test ที่ระดับ shell

ดู [[project_oc2plus_liff_shell_is_the_line_entry]] [[project_oc2plus_member_react_migration]] [[project_oc4511_4514_ux_cluster]]
