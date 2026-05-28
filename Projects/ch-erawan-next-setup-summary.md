# ch-erawan-next Setup Summary

**Last Updated:** 2026-05-26 (10:45)
**Project:** ช.เอราวัณ ออโต้ กรุ๊ป (Car Dealership Website)
**Stack:** Next.js 15 + Notion CMS + Better Auth + Turso + Cloudinary
**Status:** ✅ Mock Data Complete - รอ Vercel env vars update แล้ว go live!

---

## Project Status Overview

| Component | Status | Notes |
|-----------|--------|-------|
| Codebase | ✅ Done | Pushed to GitHub |
| Notion Schema | ✅ Done | 5 databases with full properties |
| Mock Data | ✅ Done | 6 cars, 4 blogs, 3 stories |
| Local .env | ✅ Done | 17 variables configured |
| Vercel Env | ⚠️ Pending | Need to update 3 DB IDs |
| Production | ⚠️ Pending | Will work after env update |

---

## สิ่งที่ทำล่าสุด (2026-05-26 03:30)

### Mock Data สร้างเสร็จแล้ว!

#### รถยนต์ (Cars) - 6 คัน
| รุ่น | Brand | Type | ราคา | Featured |
|------|-------|------|------|----------|
| HAVAL H6 HEV 2024 | HAVAL | SUV | 1.19-1.39M | ⭐ |
| ORA Good Cat 2024 | ORA | EV | 0.99-1.16M | ⭐ |
| TANK 500 HEV 2024 | TANK | SUV | 2.49-2.89M | ⭐ |
| GWM Cannon 2024 | GWM | Pickup | 0.86-1.10M | ⭐ |
| HAVAL Jolion HEV 2024 | HAVAL | SUV | 0.90-1.10M | ⭐ |
| ORA 03 2024 | ORA | EV | 1.10-1.30M | - |

#### บทความ (Blog) - 4 บทความ
1. **เปรียบเทียบ HAVAL H6 vs คู่แข่ง** - Category: review
2. **ORA Good Cat: รถยนต์ไฟฟ้าที่น่ารักที่สุด** - Category: review
3. **โปรโมชั่นพิเศษ! ดาวน์ 0% ผ่อนยาว 84 เดือน** - Category: promotion
4. **5 เคล็ดลับดูแลรักษารถ EV** - Category: tips

#### รีวิวลูกค้า (Stories) - 3 รีวิว
1. คุณสมชาย ใจดี - HAVAL H6 HEV ⭐⭐⭐⭐⭐
2. คุณมานี รักรถ - ORA Good Cat ⭐⭐⭐⭐⭐
3. คุณวิชัย พาณิชย์ - GWM Cannon ⭐⭐⭐⭐

---

## Environment Variables (17 ตัว)

### Notion Setup (ค่าล่าสุดที่ถูกต้อง!)
| Variable | Value |
|----------|-------|
| `NOTION_API_KEY` | `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` |
| `NOTION_CARS_DB_ID` | `36a604e0d99a8068b518f26cf65cd05c` ⚠️ อัพเดต Vercel! |
| `NOTION_BLOG_DB_ID` | `36a604e0d99a80dbbfced00ab2d89147` ⚠️ อัพเดต Vercel! |
| `NOTION_STORIES_DB_ID` | `36a604e0d99a801288f0c845f33fba91` ✅ |
| `NOTION_APPOINTMENTS_DB_ID` | `36a604e0d99a80cc8c6fd030c325a971` |
| `NOTION_CONTACTS_DB_ID` | `3159ca9915104374adc0fa19b45b6ede` |

### Auth & Security
| Variable | Value |
|----------|-------|
| `BETTER_AUTH_SECRET` | `XLEFxJbUK7yuWXAz2dML8xCgGRZytcS34aL1ltVMvYE=` |
| `BETTER_AUTH_URL` | `https://ch-erawanwebsite.vercel.app` |
| `REVALIDATE_SECRET` | `fOW4BvD62t0EPQ6IqZKRpBfg7jY5iOpXj9cgp1vhot4=` |

### Database (Turso)
| Variable | Value |
|----------|-------|
| `TURSO_DATABASE_URL` | `libsql://ch-erawan-next-cherawansystem.aws-ap-northeast-1.turso.io` |
| `TURSO_AUTH_TOKEN` | (stored in .env.local) |

### Cloudinary (Image Storage)
| Variable | Value |
|----------|-------|
| `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME` | `n5llrdnq` |
| `CLOUDINARY_API_KEY` | `748645926412145` |
| `CLOUDINARY_API_SECRET` | (stored in .env.local) |

### Google Maps
| Variable | Value |
|----------|-------|
| `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` | `AIzaSyAkVJneCSb_Ich_BaG5gCGrsoanPpXg7pE` |

### Upstash Redis
- `UPSTASH_REDIS_REST_URL`
- `UPSTASH_REDIS_REST_TOKEN`

---

## ⚠️ Vercel Environment Variables (CRITICAL)

ต้องอัพเดต IDs ใน Vercel Dashboard → Settings → Environment Variables:

| Variable | New Value (copy นี้!) |
|----------|-----------|
| `NOTION_CARS_DB_ID` | `36a604e0d99a8068b518f26cf65cd05c` |
| `NOTION_BLOG_DB_ID` | `36a604e0d99a80dbbfced00ab2d89147` |

**หลังแก้ต้อง Redeploy!**

### Debug Endpoint
ใช้ตรวจสอบ Notion connection:
https://ch-erawanwebsite.vercel.app/api/debug-notion

---

## Git & Deployment

| Item | Value |
|------|-------|
| Local folder | `/Users/nunt/ch-erawan-next` |
| GitHub repo | `https://github.com/nuntnet/ch-erawan-next-website` |
| Vercel project | `ch-erawanwebsite` |
| Production URL | https://ch-erawanwebsite.vercel.app |
| Branch | `master` |

---

## Notion Database Schema (ครบแล้ว!)

### รถยนต์ (Cars) - 17 Properties
- **Database ID:** `36a604e0d99a8068b518f26cf65cd05c`
- **Data Source:** `collection://36a604e0-d99a-8062-b8ed-000bc245f4e7`
- **Properties:** Name, Brand (GWM/HAVAL/ORA/TANK), Model, Year, Type (SUV/Sedan/Pickup/EV/Hybrid), Condition (new/used), Price Min, Price Max, Fuel Type, Transmission, Engine Size, Description, Specs (JSON), Image URLs, Video URL, Slug, Is Featured, Is Active

### บทความ (Blog) - 11 Properties
- **Database ID:** `36a604e0d99a80dbbfced00ab2d89147`
- **Data Source:** `collection://36a604e0-d99a-806b-8343-000b224fc26b`
- **Properties:** Title, Slug, Excerpt, Cover Image URL, Category (news/review/tips/promotion/company), Tags (multi-select), Author Name, Published At, Is Published, SEO Title, SEO Description

### รีวิวลูกค้า (Stories) - 10 Properties
- **Database ID:** `36a604e0d99a801288f0c845f33fba91`
- **Data Source:** `collection://36a604e0-d99a-80c4-8424-000b3372c09a`
- **Properties:** Name, Customer Name, Story, Car Model, Rating, Image URL, Email, Phone, Status (pending/approved/rejected), Is Public

---

## Code Fixes Applied

### drizzle.config.ts
- **ปัญหา:** drizzle-kit v0.31.x ไม่มี `defineConfig` export
- **แก้ไข:** เปลี่ยนเป็น `export default { ... } satisfies Config`

### Typography (IBM Plex Sans Thai + Inter)
- **Setup:** ใน layout.tsx + tailwind.config.ts ถูกต้องแล้ว

### Footer Dead Links
- **Facebook:** เก็บไว้
- **LINE:** disabled (ยังไม่มี LINE OA)
- **YouTube/Instagram:** ลบออก (ยังไม่มี)

---

## Next Steps (After Vercel Fix)

1. **Verify Site Works** - เปิด https://ch-erawanwebsite.vercel.app ดูว่า content แสดงถูกต้อง
2. **Replace Placeholder Images** - เปลี่ยนรูป Unsplash เป็นรูปรถจริงจาก Cloudinary
3. **Add Real Content** - เพิ่มรถและบทความจริงใน Notion
4. **Setup Domain** - เชื่อม custom domain (ch-erawan.com)
5. **Configure Analytics** - ใส่ Google Analytics / Vercel Analytics

---

## Notes

- **Port 3000** บนเครื่อง Nuntawit ถูกใช้โดย Hermes UI
- **Port 3001** ใช้สำหรับ ch-erawan-next dev server
- **RAM issue:** ไม่ควรรัน dev server บนเครื่องนี้ — ใช้ Vercel URL แทน

---

## Documentation

- **Sellsuki Docs:** https://docs.sellsuki.com/doc/ch-erawan-gwm-ch-erawancom-o8RTfBRGzW
- **Obsidian Note:** ~/SecondBrain/Projects/ch-erawan-next-setup-summary.md

---

#ch-erawan #vercel #notion #nextjs #project-setup
