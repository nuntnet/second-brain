# ch-erawan-next Documentation Update

**สำหรับอัพเดตเข้า:** https://docs.sellsuki.com/doc/ch-erawan-gwm-ch-erawancom-o8RTfBRGzW

---

## Project Overview

**ch-erawan-next** คือเว็บไซต์สำหรับ ช.เอราวัณ ออโต้ กรุ๊ป (GWM Dealer) สร้างด้วย Next.js 15 + Notion CMS

### Tech Stack
- **Frontend:** Next.js 15 (App Router)
- **CMS:** Notion API
- **Auth:** Better Auth
- **Database:** Turso (LibSQL)
- **Images:** Cloudinary
- **Maps:** Google Maps API
- **Cache:** Upstash Redis
- **Deploy:** Vercel

---

## URLs

| Environment | URL |
|-------------|-----|
| Production | https://ch-erawanwebsite.vercel.app |
| GitHub | https://github.com/nuntnet/ch-erawan-next-website |
| Vercel Dashboard | https://vercel.com/cherawan-next-website-s-projects/ch-erawanwebsite |

---

## Notion Databases

### 1. รถยนต์ (Cars)
- **ID:** `36a604e0d99a8068b518f26cf65cd05c`
- **Properties:** Name, Brand, Model, Year, Type, Condition, Price Min/Max, Fuel Type, Transmission, Engine Size, Description, Specs (JSON), Image URLs, Video URL, Slug, Is Featured, Is Active
- **Brand Options:** GWM, HAVAL, ORA, TANK, Other
- **Type Options:** SUV, Sedan, Pickup, EV, Hybrid

### 2. บทความ (Blog)
- **ID:** `36a604e0d99a80dbbfced00ab2d89147`
- **Properties:** Title, Slug, Excerpt, Cover Image URL, Category, Tags, Author Name, Published At, Is Published, SEO Title, SEO Description
- **Category Options:** news, review, tips, promotion, company

### 3. รีวิวลูกค้า (Customer Stories)
- **ID:** `36a604e0d99a801288f0c845f33fba91`
- **Properties:** Name, Customer Name, Story, Car Model, Rating, Image URL, Email, Phone, Status, Is Public
- **Status Options:** pending, approved, rejected

### 4. นัดหมาย (Appointments)
- **ID:** `36a604e0d99a80cc8c6fd030c325a971`
- **Properties:** Name, Customer Name, Type, Status, Phone, Email, Car Model, Branch, Preferred Date/Time, Notes

### 5. ติดต่อ (Contacts)
- **ID:** `3159ca9915104374adc0fa19b45b6ede`

---

## Environment Variables (17 ตัว)

### Notion
```
NOTION_API_KEY=<NOTION_API_TOKEN_REMOVED — store in .env, not in notes>
NOTION_CARS_DB_ID=36a604e0d99a8068b518f26cf65cd05c
NOTION_BLOG_DB_ID=36a604e0d99a80dbbfced00ab2d89147
NOTION_STORIES_DB_ID=36a604e0d99a801288f0c845f33fba91
NOTION_APPOINTMENTS_DB_ID=36a604e0d99a80cc8c6fd030c325a971
NOTION_CONTACTS_DB_ID=3159ca9915104374adc0fa19b45b6ede
```

### Auth
```
BETTER_AUTH_SECRET=XLEFxJbUK7yuWXAz2dML8xCgGRZytcS34aL1ltVMvYE=
BETTER_AUTH_URL=https://ch-erawanwebsite.vercel.app
REVALIDATE_SECRET=fOW4BvD62t0EPQ6IqZKRpBfg7jY5iOpXj9cgp1vhot4=
```

### Database
```
TURSO_DATABASE_URL=libsql://ch-erawan-next-cherawansystem.aws-ap-northeast-1.turso.io
TURSO_AUTH_TOKEN=[stored in Vercel]
```

### Cloudinary
```
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=n5llrdnq
CLOUDINARY_API_KEY=748645926412145
CLOUDINARY_API_SECRET=[stored in Vercel]
```

### Google Maps
```
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=AIzaSyAkVJneCSb_Ich_BaG5gCGrsoanPpXg7pE
```

### Upstash Redis
```
UPSTASH_REDIS_REST_URL=[stored in Vercel]
UPSTASH_REDIS_REST_TOKEN=[stored in Vercel]
```

---

## Sample Data Created

### รถยนต์ (6 คัน)
1. HAVAL H6 HEV 2024 - SUV Hybrid - 1.19-1.39M ⭐ Featured
2. ORA Good Cat 2024 - EV - 0.99-1.16M ⭐ Featured
3. TANK 500 HEV 2024 - SUV Hybrid - 2.49-2.89M ⭐ Featured
4. GWM Cannon 2024 - Pickup Diesel - 0.86-1.10M ⭐ Featured
5. HAVAL Jolion HEV 2024 - SUV Hybrid - 0.90-1.10M ⭐ Featured
6. ORA 03 2024 - EV - 1.10-1.30M

### บทความ (4 บทความ)
1. เปรียบเทียบ HAVAL H6 vs คู่แข่ง (review)
2. ORA Good Cat: รถยนต์ไฟฟ้าที่น่ารักที่สุด (review)
3. โปรโมชั่นพิเศษ! ดาวน์ 0% ผ่อนยาว 84 เดือน (promotion)
4. 5 เคล็ดลับดูแลรักษารถ EV (tips)

### รีวิวลูกค้า (3 รีวิว)
1. คุณสมชาย ใจดี - HAVAL H6 HEV ⭐⭐⭐⭐⭐
2. คุณมานี รักรถ - ORA Good Cat ⭐⭐⭐⭐⭐
3. คุณวิชัย พาณิชย์ - GWM Cannon ⭐⭐⭐⭐

---

## Troubleshooting

### Debug Endpoint
```
GET /api/debug-notion
```
Returns status of each Notion database connection.

### Common Issues

1. **"Could not find database"**
   - ตรวจสอบว่า Database ID ถูกต้อง
   - ตรวจสอบว่า Notion integration "ch-erawan-next" ได้ connect กับ database แล้ว

2. **Build Error: defineConfig**
   - drizzle-kit v0.31.x ไม่มี `defineConfig`
   - ใช้ `export default { ... } satisfies Config` แทน

---

## Development Notes

- **Port 3000:** Reserved for Hermes UI
- **Port 3001:** ch-erawan-next dev server
- **RAM Warning:** ไม่ควรรัน local dev server — ใช้ Vercel preview URL แทน

---

## Last Updated
2026-05-26 03:30 by Nuntawit
