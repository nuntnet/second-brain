# Design Improvements Implementation Plan

## Overview
Polish the website with modern UI patterns: glassmorphism cards, dark mode toggle, smooth micro-interactions (Framer Motion), and upgrade deprecated Google Maps API to AdvancedMarkerElement.

**Complexity**: Medium-High (5-8 hours)  
**Priority**: Medium  
**Blockers**: Typography setup ✅ | DS3 available ✅ | Framer Motion installed ✅

---

## Acceptance Criteria
- [ ] Card components have glassmorphism effect:
  - `backdrop-blur-xl bg-white/80 border border-white/20`
  - Subtle shadow for depth
  - Hover: slightly brighter background
- [ ] Dark mode toggle visible in header/navbar
- [ ] Dark mode preference persists (localStorage)
- [ ] All colors have dark mode equivalents:
  - Light: #0F172A background → Dark: #1a202c
  - Light: white text → Dark: #e2e8f0
  - Accent remains #DD5259 (good contrast both modes)
- [ ] Framer Motion animations:
  - Page transitions smooth (fade-in on load)
  - Cards stagger-animate on entry
  - Hover effects on buttons/links
  - Scroll reveal for below-the-fold sections
- [ ] Google Maps uses AdvancedMarkerElement (not deprecated Marker)
- [ ] No console warnings about deprecated APIs
- [ ] Performance maintained (Lighthouse >= 90)

---

## What Changes

### 1. Glassmorphism Cards
**Files**: `components/Card.tsx`, any `Card` component variants
```tsx
// Before
<div className="bg-white border border-gray-100 rounded-lg shadow-md">

// After
<div className="backdrop-blur-xl bg-white/80 border border-white/20 rounded-2xl shadow-xl hover:bg-white/90 transition-colors">
```

### 2. Dark Mode System
**Files**:
- `app/layout.tsx` — Add dark mode provider/context
- `globals.css` — Dark mode color variables
- `components/Navbar.tsx` — Add dark mode toggle button
- `tailwind.config.ts` — Ensure dark mode enabled

**Implementation**:
```tsx
// themes.ts
export const lightTheme = {
  bg: '#ffffff',
  text: '#0F172A',
  accent: '#DD5259',
  cardBg: 'rgba(255, 255, 255, 0.8)',
}

export const darkTheme = {
  bg: '#1a202c',
  text: '#e2e8f0',
  accent: '#DD5259',
  cardBg: 'rgba(26, 32, 44, 0.8)',
}
```

### 3. Framer Motion Animations
**Files**:
- `components/HeroSection.tsx` — Page fade-in
- `components/CarCard.tsx` — Stagger animation
- `components/BlogCard.tsx` — Stagger animation
- `components/CustomerStory.tsx` — Scroll reveal

**Example Animation**:
```tsx
import { motion } from 'framer-motion'

const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { staggerChildren: 0.1 }
  }
}

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 }
}

<motion.div variants={containerVariants} initial="hidden" animate="visible">
  {items.map(item => (
    <motion.div key={item.id} variants={itemVariants}>
      {item.content}
    </motion.div>
  ))}
</motion.div>
```

### 4. Google Maps AdvancedMarkerElement Migration
**Files**: `components/Map.tsx` or wherever Google Maps is used

**Before** (deprecated):
```tsx
const marker = new google.maps.Marker({
  position: { lat: 13.7563, lng: 100.5018 },
  map: map,
})
```

**After** (new):
```tsx
const { AdvancedMarkerElement } = await google.maps.importLibrary("marker")

const marker = new AdvancedMarkerElement({
  position: { lat: 13.7563, lng: 100.5018 },
  map: map,
  title: "Dealership Location"
})
```

---

## Implementation Notes

### Phase 1: Glassmorphism (1-1.5 hours)
1. Identify all card components in project
2. Update TailwindCSS classes:
   - `bg-white` → `bg-white/80 backdrop-blur-xl`
   - `border border-gray-100` → `border border-white/20`
   - Add hover states
3. Test all cards in light and dark contexts
4. Verify shadows render correctly

### Phase 2: Dark Mode Setup (1.5-2 hours)
1. Create `lib/theme.ts` with color tokens
2. Install/configure dark mode provider (use `next-themes` or Context API)
3. Add dark mode toggle button to navbar
4. Update `globals.css` with CSS variables:
   ```css
   :root {
     --bg-primary: #ffffff;
     --text-primary: #0F172A;
   }
   
   @media (prefers-color-scheme: dark) {
     :root {
       --bg-primary: #1a202c;
       --text-primary: #e2e8f0;
     }
   }
   ```
5. Update all components to use CSS variables instead of hardcoded colors
6. Test toggle persistence (localStorage)

### Phase 3: Framer Motion Animations (1.5-2 hours)
1. Framer Motion already installed ✅
2. Add page-level fade-in animation to layout
3. Add stagger animations to:
   - Car cards on homepage
   - Blog cards on blog page
   - Story cards on stories section
4. Add scroll-reveal for sections below fold
5. Add button hover animations
6. Test animations on slower devices (performance)

### Phase 4: Google Maps Migration (1 hour)
1. Find all Google Maps references
2. Update API call to include `marker` library
3. Replace `google.maps.Marker` with `AdvancedMarkerElement`
4. Test map renders correctly
5. Verify no console warnings

### Phase 5: Testing & Polish (0.5-1 hour)
- Run Lighthouse audit (target 90+)
- Test dark/light mode toggle
- Test on mobile/tablet
- Check animation performance on slow 3G
- Verify no accessibility regressions

---

## Success Metrics
✅ All cards have glassmorphism effect  
✅ Dark mode toggle works and persists  
✅ Animations smooth at 60fps  
✅ Google Maps uses new API (no warnings)  
✅ Lighthouse Performance >= 90  
✅ No visual regressions on light/dark modes  
✅ Accessibility score maintained >= 90  

---

## Blockers/Dependencies
- ✅ Framer Motion installed
- ✅ Tailwind CSS configured
- ✅ Google Maps API key available
- ⏳ May need to install `next-themes` for dark mode

---

## Optional Enhancements (Future)
- Add theme switcher component (system/light/dark)
- Custom cursor animations
- Parallax scrolling effects
- Loading skeleton animations
- Toast notifications with Framer Motion

---

## Next Steps
1. Start with glassmorphism (easiest, visible immediately)
2. Implement dark mode system
3. Add Framer Motion animations progressively
4. Upgrade Google Maps API
5. Deploy and monitor performance
6. Gather user feedback on new visual style
