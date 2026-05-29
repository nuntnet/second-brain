# Content Population Implementation Plan

## Overview
Populate the website with real dealership data: 10+ GWM/ORA cars with images, 5+ blog articles about maintenance/buying guides, and 5+ customer testimonials from real buyers.

**Complexity**: High (8-12 hours)  
**Priority**: High  
**Blockers**: Admin panel ready ✅ | Cloudinary integration ✅ | Notion write access ✅

---

## Acceptance Criteria
- [ ] 10+ cars in Cars database with:
  - Real GWM/ORA/TANK model data (name, model, year, type)
  - Price ranges from actual dealership
  - Real or high-quality placeholder images (Cloudinary URLs)
  - Is_Featured flags set correctly (4-5 cars featured)
  - Created At timestamps realistic
- [ ] 5+ blog posts in Blog database:
  - Real articles (maintenance tips, buying guides, new model announcements)
  - Cover images on Cloudinary
  - Proper categories assigned
  - Is_Published = true
  - SEO-friendly slugs generated
- [ ] 5+ customer stories:
  - Real or realistic customer names
  - 3-5 star ratings
  - Testimonial text (50-150 words)
  - Associated car model
  - Is_Public = true
- [ ] All images hosted on Cloudinary (not local/placeholder)
- [ ] Website displays all data correctly with no missing images
- [ ] Performance: page load < 2s on 4G

---

## What Changes

### Cars Data (10 minimum):
```
Example structure:
- Model: TANK 500 | Year: 2024 | Price: 899k-1.2M | Type: SUV | Featured: true
- Model: TANK 300 | Year: 2024 | Price: 699k-899k | Type: SUV | Featured: true
- Model: HAVAL H6 HEV | Year: 2024 | Price: 799k-1M | Type: SUV Hybrid | Featured: true
- Model: ORA Good Cat | Year: 2024 | Price: 549k-699k | Type: Hatchback EV | Featured: true
- [6 more vehicles with real specs]
```

### Blog Articles (5 minimum):
- "วิธีบำรุงรักษารถยนต์ให้อยู่ยาว" (Vehicle Maintenance Tips)
- "คำแนะนำในการเลือกรถ SUV ที่เหมาะสม" (Guide to Choosing an SUV)
- "เทคโนโลยีไฮบริดของ GWM คืออะไร" (GWM Hybrid Technology Explained)
- "ข้อมูลราคารถยนต์ประจำปี 2024" (2024 Car Pricing Guide)
- "รีวิว ORA Good Cat: รถไฟฟ้าราคาประหยัด" (ORA Good Cat Review)

### Customer Stories (5 minimum):
- Customer names + ratings + short stories
- Associated with specific car models
- Realistic testimonials (150-200 words each)

---

## Implementation Notes

### Phase 1: Prepare Image Assets (2-3 hours)
- Source car images:
  - GWM official website / media kit
  - Unsplash high-quality auto photos (fallback)
  - Dealership photos if available
- Resize/optimize for web (1200x800 min)
- Upload to Cloudinary via UI or API
- Get Cloudinary URLs

### Phase 2: Create Car Records (1.5-2 hours)
- Use Admin Panel created in Phase 1
- Input 10+ cars with:
  - Brand, Model, Year, Type, Price Min/Max
  - Cloudinary image URL
  - Is_Featured (yes/no)
  - Verified real specs from official sources

### Phase 3: Write & Publish Blog Articles (2-3 hours)
- Write 5 blog articles (500-1000 words each)
- Include relevant keywords for SEO
- Add cover images (Cloudinary)
- Publish via Admin Panel

### Phase 4: Collect Customer Stories (1.5-2 hours)
- Create or gather 5 realistic customer testimonials
- Assign star ratings (4-5 stars typical)
- Link to specific car models
- Publish as Public stories

### Phase 5: Verify & Optimize (1 hour)
- Check all images load correctly
- Verify page performance (Lighthouse)
- Test responsive design (mobile/tablet/desktop)
- Confirm no broken links or missing data

---

## Success Metrics
✅ Website shows 10+ cars with images on homepage  
✅ Blog page displays 5+ articles with preview cards  
✅ Stories section shows 5+ testimonials  
✅ All images from Cloudinary (fast CDN delivery)  
✅ Homepage Lighthouse score >= 90  
✅ No 404 errors for any images/content  

---

## Blockers/Dependencies
- ✅ Admin panel CRUD working
- ✅ Cloudinary integration configured
- ✅ Notion database write access
- ⏳ Need actual car data (specs, pricing) — can use official GWM websites
- ⏳ Need customer testimonial content — can be realistic placeholders

---

## Next Steps
1. Use Admin Panel to start adding cars
2. Upload images to Cloudinary as you go
3. Write blog articles in Notion or via Admin Panel
4. Create customer stories
5. Deploy and monitor performance
6. Get real customer testimonials as feedback arrives
