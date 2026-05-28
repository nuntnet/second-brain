# Ch-Erawan Notion Database Setup Guide

Created: 2026-05-26
Project: ch-erawan-next

## Problem
Notion databases มีแค่ column "Name" — ต้องเพิ่ม properties ตามที่ code ต้องการ

## Database IDs (ถูกต้องแล้ว)
- Cars: `36a604e0d99a8068b518f26cf65cd05c`
- Blog: `36a604e0d99a80dbbfced00ab2d89147`
- Stories: `36a604e0d99a801288f0c845f33fba91`

---

## 1. Cars Database — Properties ที่ต้องเพิ่ม

| Property Name | Type | Options/Notes |
|--------------|------|---------------|
| Name | Title | ✅ มีอยู่แล้ว |
| Brand | Select | Mazda, Ford, Mitsubishi, GWM, Deepal, Kia |
| Model | Text | เช่น CX-5, Ranger, Triton |
| Year | Number | เช่น 2024 |
| Type | Select | sedan, suv, pickup, hatchback, mpv, ev, other |
| Condition | Select | new, used |
| Price Min | Number | ราคาเริ่มต้น |
| Price Max | Number | ราคาสูงสุด |
| Engine Size | Text | เช่น 2.0L, 2.5L |
| Transmission | Select | auto, manual |
| Fuel Type | Select | petrol, diesel, hybrid, electric |
| Description | Text | คำอธิบายรถ |
| Specs | Text | JSON string (optional) |
| Image URLs | Text | URLs คั่นด้วย newline |
| Video URL | URL | YouTube link (optional) |
| Is Active | Checkbox | ✓ = แสดงบนเว็บ |
| Is Featured | Checkbox | ✓ = แสดงในหน้าแรก |
| Slug | Text | URL-friendly name |

---

## 2. Blog Database — Properties ที่ต้องเพิ่ม

| Property Name | Type | Options/Notes |
|--------------|------|---------------|
| Title | Title | เปลี่ยนจาก Name → Title |
| Slug | Text | URL-friendly, เช่น "mazda-cx5-review" |
| Excerpt | Text | สรุปสั้นๆ 1-2 ประโยค |
| Cover Image URL | Text | Cloudinary/Unsplash URL |
| Category | Select | review, tips, news, promotion, csr |
| Tags | Multi-select | รถยนต์, โปรโมชั่น, ข่าวสาร, เคล็ดลับ, CSR |
| SEO Title | Text | (optional) |
| SEO Description | Text | (optional) |
| Is Published | Checkbox | ✓ = เผยแพร่ |
| Published At | Date | วันที่เผยแพร่ |
| Author Name | Text | ชื่อผู้เขียน |

---

## 3. Stories Database — Properties ที่ต้องเพิ่ม

| Property Name | Type | Options/Notes |
|--------------|------|---------------|
| Name | Title | ✅ มีอยู่แล้ว (ใช้เป็น ID) |
| Customer Name | Text | ชื่อลูกค้า |
| Email | Email | อีเมลลูกค้า |
| Phone | Phone | เบอร์โทร |
| Story | Text | เรื่องราว/รีวิว |
| Rating | Number | 1-5 |
| Car Model | Text | รุ่นรถที่ซื้อ |
| Image URL | URL | รูปลูกค้า (optional) |
| Status | Select | pending, approved, rejected |
| Is Public | Checkbox | ✓ = แสดงบนเว็บ |

---

## Mock Data — Cars (พร้อม copy)

### Car 1: Mazda CX-5
- **Name:** Mazda CX-5 2024
- **Brand:** Mazda
- **Model:** CX-5
- **Year:** 2024
- **Type:** suv
- **Condition:** new
- **Price Min:** 1290000
- **Price Max:** 1699000
- **Engine Size:** 2.0L SKYACTIV-G
- **Transmission:** auto
- **Fuel Type:** petrol
- **Description:** SUV ขนาดกลางที่โดดเด่นด้วยดีไซน์ KODO และเทคโนโลยี SKYACTIV ให้ทั้งความประหยัดและสมรรถนะที่เหนือชั้น
- **Image URLs:** https://images.unsplash.com/photo-1609521263047-f8f205293f24?w=800&q=80
- **Is Active:** ✓
- **Is Featured:** ✓
- **Slug:** mazda-cx5-2024

### Car 2: Ford Ranger Wildtrak
- **Name:** Ford Ranger Wildtrak 2024
- **Brand:** Ford
- **Model:** Ranger Wildtrak
- **Year:** 2024
- **Type:** pickup
- **Condition:** new
- **Price Min:** 1099000
- **Price Max:** 1399000
- **Engine Size:** 2.0L Bi-Turbo
- **Transmission:** auto
- **Fuel Type:** diesel
- **Description:** กระบะพันธุ์แกร่ง ออกแบบมาเพื่อการผจญภัย พร้อมเทคโนโลยีล้ำสมัยและความสะดวกสบายระดับพรีเมียม
- **Image URLs:** https://images.unsplash.com/photo-1551830820-c6874016b5f0?w=800&q=80
- **Is Active:** ✓
- **Is Featured:** ✓
- **Slug:** ford-ranger-wildtrak-2024

### Car 3: Mitsubishi Triton
- **Name:** Mitsubishi Triton 2024
- **Brand:** Mitsubishi
- **Model:** Triton
- **Year:** 2024
- **Type:** pickup
- **Condition:** new
- **Price Min:** 599000
- **Price Max:** 1199000
- **Engine Size:** 2.4L MIVEC
- **Transmission:** auto
- **Fuel Type:** diesel
- **Description:** กระบะที่แข็งแกร่งและทนทาน เหมาะสำหรับทั้งการใช้งานและการผจญภัย
- **Image URLs:** https://images.unsplash.com/photo-1556909114-f6e7ad7d3136?w=800&q=80
- **Is Active:** ✓
- **Is Featured:** ✓
- **Slug:** mitsubishi-triton-2024

### Car 4: GWM Tank 500
- **Name:** GWM Tank 500 2024
- **Brand:** GWM
- **Model:** Tank 500
- **Year:** 2024
- **Type:** suv
- **Condition:** new
- **Price Min:** 2499000
- **Price Max:** 2799000
- **Engine Size:** 3.0L V6 Turbo
- **Transmission:** auto
- **Fuel Type:** petrol
- **Description:** SUV หรูระดับพรีเมียม 7 ที่นั่ง พร้อมระบบขับเคลื่อน 4 ล้ออัจฉริยะ
- **Image URLs:** https://images.unsplash.com/photo-1558618666-fcd25c85cd64?w=800&q=80
- **Is Active:** ✓
- **Is Featured:** ✓
- **Slug:** gwm-tank-500-2024

### Car 5: Deepal S07
- **Name:** Deepal S07 2024
- **Brand:** Deepal
- **Model:** S07
- **Year:** 2024
- **Type:** ev
- **Condition:** new
- **Price Min:** 1199000
- **Price Max:** 1399000
- **Engine Size:** Electric
- **Transmission:** auto
- **Fuel Type:** electric
- **Description:** SUV ไฟฟ้ารุ่นใหม่ล่าสุด ดีไซน์ล้ำสมัย วิ่งได้ไกลถึง 520 กม.
- **Image URLs:** https://images.unsplash.com/photo-1593941707882-a5bba14938c7?w=800&q=80
- **Is Active:** ✓
- **Is Featured:** ✓
- **Slug:** deepal-s07-2024

### Car 6: Kia Sportage
- **Name:** Kia Sportage 2024
- **Brand:** Kia
- **Model:** Sportage
- **Year:** 2024
- **Type:** suv
- **Condition:** new
- **Price Min:** 1499000
- **Price Max:** 1799000
- **Engine Size:** 1.6L Turbo
- **Transmission:** auto
- **Fuel Type:** hybrid
- **Description:** SUV ดีไซน์ล้ำสมัย เทคโนโลยีครบครัน พร้อมระบบไฮบริดประหยัดน้ำมัน
- **Image URLs:** https://images.unsplash.com/photo-1617788138017-80ad40651399?w=800&q=80
- **Is Active:** ✓
- **Is Featured:** ✓
- **Slug:** kia-sportage-2024

---

## Mock Data — Blog Posts (พร้อม copy)

### Post 1: Mazda CX-5 Review
- **Title:** รีวิว Mazda CX-5 2024 — SUV ที่ลงตัวทุกด้าน
- **Slug:** mazda-cx5-2024-review
- **Excerpt:** สัมผัสประสบการณ์ขับขี่ที่เหนือระดับกับ Mazda CX-5 ใหม่ ดีไซน์ KODO สะกดทุกสายตา
- **Cover Image URL:** https://images.unsplash.com/photo-1609521263047-f8f205293f24?w=800&q=80
- **Category:** review
- **Tags:** รถยนต์
- **Is Published:** ✓
- **Published At:** 2024-05-15
- **Author Name:** ทีมงาน ช.เอราวัณ

### Post 2: โปรโมชั่น Ford Ranger
- **Title:** โปรโมชั่นพิเศษ Ford Ranger — ดาวน์ 0% ผ่อนยาว 84 เดือน
- **Slug:** ford-ranger-promotion-2024
- **Excerpt:** โอกาสทองในการเป็นเจ้าของ Ford Ranger ด้วยข้อเสนอพิเศษสุดคุ้ม!
- **Cover Image URL:** https://images.unsplash.com/photo-1551830820-c6874016b5f0?w=800&q=80
- **Category:** promotion
- **Tags:** โปรโมชั่น, รถยนต์
- **Is Published:** ✓
- **Published At:** 2024-05-20
- **Author Name:** ทีมงาน ช.เอราวัณ

### Post 3: เคล็ดลับดูแลรถ
- **Title:** 5 เคล็ดลับดูแลรถยนต์หน้าฝน ให้รถพร้อมใช้งานเสมอ
- **Slug:** car-care-tips-rainy-season
- **Excerpt:** เตรียมพร้อมรับมือหน้าฝนด้วยเคล็ดลับง่ายๆ ที่ช่วยให้รถของคุณปลอดภัย
- **Cover Image URL:** https://images.unsplash.com/photo-1503376780353-7e6692767b70?w=800&q=80
- **Category:** tips
- **Tags:** เคล็ดลับ
- **Is Published:** ✓
- **Published At:** 2024-05-22
- **Author Name:** ทีมงาน ช.เอราวัณ

---

## Mock Data — Customer Stories (พร้อม copy)

### Story 1
- **Name:** story-001
- **Customer Name:** คุณสมชาย ใจดี
- **Story:** ประทับใจมากครับ พนักงานให้บริการดีมาก อธิบายรายละเอียดรถชัดเจน ตัดสินใจซื้อ Mazda CX-5 ไม่ผิดหวังเลย ขับดีมาก ประหยัดน้ำมันกว่าที่คิด
- **Rating:** 5
- **Car Model:** Mazda CX-5
- **Status:** approved
- **Is Public:** ✓

### Story 2
- **Name:** story-002
- **Customer Name:** คุณนภา รักษาสุข
- **Story:** ซื้อ Ford Ranger มาใช้งานได้ 2 ปีแล้ว ประทับใจบริการหลังการขายมาก เข้าศูนย์ทุกครั้งได้รับการดูแลดี
- **Rating:** 5
- **Car Model:** Ford Ranger
- **Status:** approved
- **Is Public:** ✓

### Story 3
- **Name:** story-003
- **Customer Name:** คุณวิชัย ก้าวหน้า
- **Story:** เปลี่ยนมาใช้ Deepal S07 รถไฟฟ้า ประหยัดค่าน้ำมันไปเยอะมาก ชาร์จที่บ้านสะดวกมาก ขอบคุณ ช.เอราวัณ ที่แนะนำดีครับ
- **Rating:** 5
- **Car Model:** Deepal S07
- **Status:** approved
- **Is Public:** ✓

---

## Brand Images — Updated URLs (Unsplash)

```typescript
export const BRAND_IMAGES: Record<string, string> = {
  Mazda:      "https://images.unsplash.com/photo-1609521263047-f8f205293f24?w=800&q=80&auto=format&fit=crop",
  Ford:       "https://images.unsplash.com/photo-1551830820-c6874016b5f0?w=800&q=80&auto=format&fit=crop",
  Mitsubishi: "https://images.unsplash.com/photo-1556909114-f6e7ad7d3136?w=800&q=80&auto=format&fit=crop",
  GWM:        "https://images.unsplash.com/photo-1558618666-fcd25c85cd64?w=800&q=80&auto=format&fit=crop",
  Deepal:     "https://images.unsplash.com/photo-1593941707882-a5bba14938c7?w=800&q=80&auto=format&fit=crop",
  Kia:        "https://images.unsplash.com/photo-1617788138017-80ad40651399?w=800&q=80&auto=format&fit=crop",
  default:    "https://images.unsplash.com/photo-1503376780353-7e6692767b70?w=800&q=80&auto=format&fit=crop",
};
```

---

## Next Steps

1. **เปิด Notion** → ไปที่แต่ละ database
2. **เพิ่ม properties** ตามตารางด้านบน
3. **เพิ่ม mock data** โดย copy จากด้านบน
4. **Redeploy Vercel** หลังจากเพิ่มข้อมูลครบ

---

## Related
- [[ch-erawan-next-setup-summary]]
- Project: ch-erawan-next
