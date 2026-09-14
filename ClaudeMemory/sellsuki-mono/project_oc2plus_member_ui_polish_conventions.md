---
name: project_oc2plus_member_ui_polish_conventions
description: กติกา UI ที่เคาะระหว่างรอบเก็บงานดีไซน์ member app (2026-09-14, MR !80/!82/!83) — สีแบรนด์ = กดได้เท่านั้น, ป้ายในกริด/แท็บใช้ caption ไม่ใช่ label, placeholder รูปต้องมีไอคอน
metadata:
  node_type: memory
  type: project
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
---

รอบเก็บงานดีไซน์ทั้งแอป 2026-09-14 (`feat/oc-4344-ui-polish` → develop, MR !80)
วัดจากเบราว์เซอร์จริงที่ 375 และ 1280 ทุกข้อ — ข้างล่างคือกติกาที่เคาะแล้วและควรใช้ต่อ

## 1. พื้นสีแบรนด์ = "กดได้" เท่านั้น
`--brand-soft` + `--brand-line` สงวนไว้ให้ปุ่มรอง · ของที่อ่านอย่างเดียว (แถบแต้มคงเหลือ
หน้าประวัติ) ต้องเป็น `--surface` + `--line` · ปุ่มหลักจริงของแอปมีตัวเดียวคือปุ่มแลกของรางวัล
(`--brand-ink` ทึบ) ส่วน "แสดงบัตรสมาชิก" ใช้ `--ink-solid`

## 2. ป้ายในกริดไอคอน/แถบแท็บ = caption ไม่ใช่ label
ช่องกว้าง ~67–74px · label (18px) ทำให้คำไทยสองพยางค์ ("ค้นหาสาขา" "ติดต่อร้าน"
"ของรางวัล" "แจ้งใบเสร็จ") ตกบรรทัดพอดีเป๊ะ · caption (15px) พอดี
**แต่ยังต้องจอง 2 บรรทัด** (`min-height: calc(2 * leading * 1em)`) เพราะ EN ยาวกว่า
("Find a branch" 100px ใน 67px) ถ้าไม่จอง ไอคอนในแถวจะหลุดเส้นฐานเดียวกัน

## 3. rail แนวนอนต้อง `flex: 1 1 <basis>` + min/max
`flex: none; width: 76px` ทำให้ 4 ช่องล้น 15px ที่ 375 → ป้ายสุดท้ายโดนตัดกลางตัว
(อ่านเป็นบั๊ก ไม่ใช่ "เลื่อนดูต่อได้") · `flex: 1 1 76px; min-width: 68px; max-width: 96px`
= ที่แคบหดจนครบจอ, ที่กว้าง (960px desktop) ไม่ยืดเวิ้งว้าง, ถ้ารายการเยอะขึ้นก็ยังเลื่อนได้

## 4. ช่องรูปว่าง = ไอคอนใต้รูป ไม่ใช่กล่องเทาเปล่า
วาง `<TicketIcon>`/`<GiftIcon>` ไว้ในกล่องแล้ววาง `<img>` ทับด้วย `position:absolute; inset:0`
→ รูปที่ไม่มี **และ** รูปที่ 404 ตกลงมาที่ placeholder เดียวกันโดยไม่ต้องจับ onError

## 5. ห้ามใช้ตัวอักษรแทนไอคอน
"⌕" ในช่องค้นหาไม่มี glyph ดี ๆ ในฟอนต์ไทย · ไอคอนทั้งหมดอยู่ที่
`react/pages/Home/icons.tsx` (วาดเอง ไม่มีปัญหา license) — เพิ่ม `SearchIcon` `GiftIcon`
`ReceiptIcon` รอบนี้ · `react/shell/icons.tsx` เป็นคนละชุด (แท็บ + สแกน) ห้ามข้ามไปหยิบ

## 6. การ์ดต้องไม่พูดสองอย่างที่ขัดกัน
TierCard เคยขึ้น "ยังไม่มีข้อมูลระดับสมาชิก" พร้อม "ระดับถัดไป: Silver"
→ `not_yet_title` + `first` เมื่อมี next แต่ไม่มี current · แถบ progress กับบรรทัด
"อีก N แต้ม" เป็นค่าเดียวกันสองมุมมอง ต้องโผล่/หายพร้อมกัน (`TierCard.spec.tsx` ล็อกไว้แล้ว)

## 7. สูตรการ์ดเดียวทั้งแอป (รอบ 2, MR !82)

`background: var(--surface)` + `border: 1px solid var(--line)` + `--radius-company-xl` + `--shadow-sm`
· พื้นหน้าแอปคือ `--surface-2` → **การ์ดห้ามใช้ `--surface-2`** ไม่งั้นกลายเป็นเส้นขอบลอยบนพื้นสีเดียวกัน
(การ์ดระดับสมาชิก + แถบแต้มเคยเป็นแบบนั้น ขณะที่การ์ดข้าง ๆ เป็นสีขาว)
· ใช้กับ: tier, points, quick-actions, play-zone, check-in, profile card/row, history balance+list, reward card

## 8. ระยะห่างเป็นของ layout ไม่ใช่ของบล็อก

`.oc2-home` เป็น flex column ที่มี `gap` เดียว · ห้ามให้ลูกถือ `margin-top` เอง
(เดิมมี 5 ค่าปนกัน 12/9/10/14/20 จังหวะเลยเปลี่ยนไปเรื่อย ๆ)

## 9. semantic token ห้ามใช้ตกแต่ง

`--warn-ink` แปลว่า "มีอะไรต้องดู" · เคยถูกใช้ทาไอคอนประกายของหัวข้อ "สนุกไปกับเรา"
→ อ่านเป็น alert และเป็นสีส้มจุดเดียวในหน้าที่เหลือเป็นม่วง

## 10. รูปที่โหลดไม่ขึ้นต้องมี fallback ไม่ใช่แค่ "ไม่มีรูป"

`{logo ? <img/> : <initial/>}` ไม่พอ — url ที่มีแต่โหลดไม่ขึ้นให้ไอคอนรูปแตกของเบราว์เซอร์
· ต้องมี `onError` ด้วย · รวมไว้ที่ `components/BrandLogoImage.tsx` ใช้ร่วมกัน 5 จอ
(login, register, point-claim ×3) และ `Home/components/HomeCatalog.tsx` วางไอคอนไว้ใต้ `<img>`

## 11. overlay ที่สูง 100vh ต้องกันที่ให้ tab bar เอง

tab bar เป็น `position: fixed` สูง 64px · `.oc2-shell__content` จองที่ไว้แล้ว แต่ overlay ที่
`min-height: 100vh` (หน้าบัตรสมาชิก) ไม่ได้จอง → บรรทัดล่างสุดอยู่ใต้แถบ อ่านไม่ได้
· วิธีตรวจ: เทียบ `getBoundingClientRect().bottom` ของ element ล่างสุดกับ `tabbar.top`

**How to apply:** ก่อนแตะ CSS ของแอปนี้ อ่าน 11 ข้อนี้ก่อน — ทุกข้อมาจากของที่พังจริงบนจอจริง
ไม่ใช่ความชอบ · baseline เทสของ repo นี้คือ **1179/1182** (3 ที่แดงคือ `memberLanguage.spec.tsx`
Node 26/jsdom พังอยู่ก่อนแล้วบน develop)

เชื่อม [[project_oc2plus_member_app_design_v2]] [[reference_oc2plus_member_frontend]] [[project_oc2plus_member_react_migration]]
