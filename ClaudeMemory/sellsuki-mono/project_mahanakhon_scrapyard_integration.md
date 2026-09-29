---
name: project-mahanakhon-scrapyard-integration
description: "มหานครโลหะ (scrap metal, 3 สาขา) เชื่อม Scrapyard POS กับ OC2Plus+BOLA — proposal V.3, การตัดสินใจ 2026-09-29, การ์ดที่ยังต้องเขียนต่อจาก OC-2275"
metadata:
  node_type: memory
  type: project
  originSessionId: 80f3d634-2df8-4993-a006-dce965f79cd9
  modified: 2026-09-29T15:53:51.691Z
---

ลูกค้า **มหานครโลหะ** ต้องการ CRM/loyalty (OC2Plus) + BOLA เชื่อมโปรแกรมรับซื้อ **Scrapyard** (proposal Google Doc `1yEeU4r1Z416d3Ny2FG80yvzkBd6OmqvUvkDEPd_alyk`, V.3). Scrapyard เป็น **สองทาง**: เรียกเรา (member import/update, earn by ยอดเงิน, active/inactive) และเราเรียกเขา (ราคารับซื้อ, ประวัติขาย, kg/carbon) + webhook Reference ID ที่เราส่งกลับ. epic OC-2275 เดิมมองแค่ทางเข้า และ `point.adjust` ไม่ใช่ purchase earn (ต้อง `purchase.submit`, OC-4429 แค่จองชื่อ).

**Decisions (user, 2026-09-29):**
- 3 สาขา = 3 company, **Scrapyard ถือ API key แยกต่อสาขา** (ไม่ต้องมี branch_code routing)
- ราคา/ประวัติขาย/kg/carbon: **ไม่เก็บ DB ไม่ sync** — get ตาม member id จาก backend Scrapyard
- Custom CMS ฝั่งสมาชิก = **โปรเจกต์แยก per-company** ไม่เอา business มหานครโลหะปนใน OC2Plus → connector Scrapyard อยู่ใน BFF ของ CMS ไม่ใช่ 3rdparty-api
- active/inactive ต้องเป็น **ฟีเจอร์ product ของ OC2Plus** ไม่ใช่เฉพาะกิจ
- PDPA เลขบัตร ปชช. ไปทาง OC-4342 (custom fields + PDPA guardrails, To Do)

**Why:** ป้องกันการ์ดซ้ำ/ผิดทิศทางตอนแตกงาน. **How to apply:** ก่อนเขียนการ์ด อ่านไฟล์นี้; การ์ด membership-from-contacts คือ **OC-4583** (Invitation Link จาก Contact/Member/Follower, In Progress) — user พิมพ์เลขผิดเป็น OC-4538 ซึ่งคือบั๊ก /v1/me/point* ใน member-api. Open: ให้ Scrapyard ส่ง kg_12m/band มาเพื่อ push ไป BOLA custom field หรือตัด segment ตาม kg ออก. Nav: ทำ generic "custom menu item" + token แลก session, ไม่ยัด tab (SHELL_TABS 4 tab).

เกี่ยวข้อง [[project_oc2275_remaining_blocked_on_decisions]] [[reference_oc2plus_member_page_registry]] [[project_oc_bola_domain_boundary]]
