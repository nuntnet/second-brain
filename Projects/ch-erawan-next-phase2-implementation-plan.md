# ch-erawan-next Phase 2 Implementation Plan

**Created:** 2026-05-29
**Project:** ช.เอราวัณ ออโต้ กรุ๊ป Website
**Status:** Planning → Review → Execution

---

## ภาพรวม (Overview)

หลังจาก Phase 1 เสร็จสมบูรณ์ (เว็บ live ที่ https://ch-erawanwebsite.vercel.app/ + Notion CMS ทำงานครบ + Forms ทำงานได้) — Phase 2 จะเน้น 4 features ใหญ่:

| # | Feature | Priority | Complexity | Est. Time |
|---|---------|----------|------------|-----------|
| 1 | Admin Panel Testing | 🟠 High | Medium | 2-3 ชม. |
| 2 | Content Population | 🟡 Medium | High | 1-2 สัปดาห์ |
| 3 | Custom Domain | 🟡 Medium | Low | 1-2 ชม. |
| 4 | Design Improvements | 🟢 Medium | Medium-High | 1 สัปดาห์ |

---

## Feature 1: Admin Panel Testing

### เป้าหมาย
ตรวจสอบว่า Admin Panel (Better Auth + Turso) สามารถ login + จัดการ content ใน Notion ได้จริง

### Acceptance Criteria
- [ ] Admin user สามารถ login ผ่าน /admin/login
- [ ] Session persistence ทำงานถูกต้อง (Turso)
- [ ] เห็น dashboard หลังจาก login
- [ ] CRUD operations ทำงาน:
  - [ ] Create: เพิ่มรถใหม่ใน Cars database
  - [ ] Read: ดูรายการรถ + บทความ + รีวิว
  - [ ] Update: แก้ไขข้อมูลรถ/บทความ
  - [ ] Delete: ลบรายการ (พร้อม confirmation)
- [ ] Logout ทำงานถูกต้อง

### ขั้นตอน
1. ตรวจสอบ Admin credentials ใน Turso DB
2. Test login flow บน https://ch-erawanwebsite.vercel.app/admin/login
3. Verify Notion API write operations
4. Test ทุก CRUD endpoint
5. รายงานปัญหา (ถ้ามี) + แก้ไข

### Dependencies
- ✅ Better Auth configured
- ✅ Turso database connected
- ✅ Notion API write access

---

## Feature 2: Content Population

### เป้าหมาย
แทนที่ Mock Data ด้วยข้อมูลจริงของ ช.เอราวัณ — รถ, รูป, บทความ, รีวิวลูกค้า

### Acceptance Criteria
- [ ] **Cars:** 10+ รุ่นจริง (GWM, ORA, HAVAL, TANK) พร้อมรูปจริงจาก Cloudinary
- [ ] **Blog:** 5+ บทความจริง (รีวิว, เปรียบเทียบ, คู่มือบำรุงรักษา, โปรโมชั่น)
- [ ] **Customer Stories:** 5+ รีวิวลูกค้าจริง (พร้อมรูปลูกค้า + รถ)
- [ ] รูปทั้งหมด optimize ผ่าน Cloudinary (WebP + responsive)

### Sub-tasks

#### 2.1 รูปภาพรถ (Cloudinary Upload)
- รวบรวมรูปรถจาก dealership (GWM/ORA/HAVAL/TANK)
- Upload ไป Cloudinary (folder: `ch-erawan/cars/`)
- Get image URLs สำหรับใส่ใน Notion

#### 2.2 ข้อมูลรถ (Notion Cars Database)
- เขียนข้อมูลรถจริงทีละรุ่น:
  - รุ่น, ปี, ราคา (min/max)
  - Specs (เครื่อง, แรงม้า, ระบบขับ)
  - Features
  - Image URLs (จาก Cloudinary)
  - Is Featured, Is Active

#### 2.3 บทความ Blog
- หัวข้อที่แนะนำ:
  1. รีวิว HAVAL H6 HEV vs Toyota Corolla Cross
  2. คู่มือเลือกซื้อรถยนต์ไฟฟ้า ORA
  3. เปรียบเทียบ TANK 300 vs TANK 500
  4. โปรโมชั่นรถ GWM ประจำเดือน
  5. การบำรุงรักษารถ EV เบื้องต้น

#### 2.4 Customer Stories
- รวบรวมรีวิวลูกค้าจริง (ขอ permission ก่อน)
- รูปลูกค้า + รถ
- Rating + Story content
- Status = approved

### Dependencies
- ✅ Cloudinary integration
- ✅ Notion databases (Cars, Blog, Stories) schema ครบ
- ⏳ ต้องการเนื้อหาจริงจาก dealership

---

## Feature 3: Custom Domain (ch-erawan.com)

### เป้าหมาย
ให้เว็บไซต์ accessible ที่ https://ch-erawan.com แทน Vercel subdomain

### Acceptance Criteria
- [ ] ch-erawan.com → resolves to Vercel production
- [ ] www.ch-erawan.com → redirects to ch-erawan.com
- [ ] HTTPS certificate auto-provisioned (Let's Encrypt via Vercel)
- [ ] อัพเดต `BETTER_AUTH_URL` ใน Vercel env vars
- [ ] อัพเดต Notion webhook URLs (ถ้ามี)

### ขั้นตอน
1. **เช็ค domain ownership** — ch-erawan.com registered ที่ไหน? (GoDaddy, Namecheap, Hostatom, etc.)
2. **เพิ่ม domain ใน Vercel** — Project Settings → Domains → Add `ch-erawan.com`
3. **Update DNS records** ที่ registrar:
   - A Record: `@` → `76.76.21.21` (Vercel IP)
   - CNAME: `www` → `cname.vercel-dns.com`
4. **รอ DNS propagate** (5 นาที - 24 ชม.)
5. **Verify HTTPS** ทำงาน
6. **Update env vars:**
   - `BETTER_AUTH_URL` = `https://ch-erawan.com`
7. **Redeploy** + test

### Dependencies
- ⏳ ยืนยัน domain ownership
- ✅ Vercel project access

---

## Feature 4: Design Improvements

### เป้าหมาย
ยกระดับ design จาก functional → impressive (สวยและ professional)

### Acceptance Criteria
- [ ] Glassmorphism effect บน Cards (Cars, Blog, Stories)
- [ ] Dark Mode toggle + persistence (localStorage)
- [ ] Framer Motion animations (scroll reveal, hover, page transitions)
- [ ] Google Maps ใช้ AdvancedMarkerElement (fix deprecation warning)
- [ ] Loading=async parameter สำหรับ Google Maps script
- [ ] Lighthouse score 90+ (Performance, Accessibility, SEO)

### Sub-tasks

#### 4.1 Glassmorphism Cards
```css
/* ก่อน */
.card { background: white; border: 1px solid #e5e7eb; }

/* หลัง */
.card {
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
}
```

#### 4.2 Dark Mode
- เพิ่ม `darkMode: 'class'` ใน tailwind.config.ts
- สร้าง ThemeToggle component (Sun/Moon icon)
- ใช้ next-themes สำหรับ SSR support
- Update color tokens ใน globals.css (light/dark variants)

#### 4.3 Framer Motion
```bash
npm install framer-motion
```
- Add scroll-reveal animations บน sections
- Page transition animations
- Card hover micro-interactions
- Hero text stagger animations

#### 4.4 Google Maps Fix
- เปลี่ยน `google.maps.Marker` → `google.maps.marker.AdvancedMarkerElement`
- เพิ่ม `loading=async` ใน script tag
- ใช้ Map ID สำหรับ Advanced Markers

### Dependencies
- ✅ Tailwind CSS configured
- ✅ Google Maps API key
- ⏳ ต้องติดตั้ง framer-motion + next-themes

---

## Sequencing & Timeline (แนะนำ)

### สัปดาห์ที่ 1
- **Day 1-2:** Feature 1 (Admin Panel Testing) — ความเสี่ยงต่ำ, เห็นผลเร็ว
- **Day 3-5:** Feature 3 (Custom Domain) — รอ DNS propagate ระหว่างทำอย่างอื่น

### สัปดาห์ที่ 2-3
- **Feature 2 (Content Population)** — ทำคู่ขนานกับ Feature 4 ได้
- **Feature 4 (Design Improvements)** — เริ่มจาก Glassmorphism → Dark Mode → Framer Motion

### Milestone
- **End of Week 1:** Admin Panel ✅ + Custom Domain ✅
- **End of Week 3:** Content จริงครบ + Design ใหม่

---

## Decision Gates

ก่อนเริ่มทำแต่ละ Feature ต้อง confirm:

### Feature 1 — Admin Panel
- [ ] ✅ พร้อมทำเลย? หรือ
- [ ] ⏸️ Skip ไปก่อน (ใช้ Notion UI โดยตรง)

### Feature 2 — Content
- [ ] ✅ มีรูปรถจริงพร้อมหรือยัง?
- [ ] ✅ จะให้ผมช่วยเขียน blog draft มั้ย?

### Feature 3 — Custom Domain
- [ ] ✅ ch-erawan.com registered ที่ไหน?
- [ ] ✅ มี access เข้า DNS settings มั้ย?

### Feature 4 — Design
- [ ] ✅ ทำทั้ง 4 sub-tasks หรือเลือก?
- [ ] ✅ Dark mode เป็น default หรือ light?

---

## Next Step

**รบกวน review เอกสารนี้** แล้วเลือก:

1. ✅ **Approve ทั้งแผน** → ผมเริ่มจาก Feature 1 (Admin Panel Testing)
2. 🔄 **แก้ไขบางจุด** → บอกผมว่าจะปรับอะไร
3. 🎯 **เริ่มจาก Feature อื่น** → เลือก Feature ที่อยากทำก่อน

---

**ตามรูปแบบการทำงานของคุณ:**
- ✅ เอกสาร MD ก่อน (เสร็จแล้ว)
- ⏳ Review + feedback
- ⏳ Mark DONE
- ⏳ Update Kanban
- ⏳ Execute
