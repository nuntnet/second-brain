---
name: reference_ssk_components_docs
description: "DS3 ssk-* web components ship their own docs inside node_modules — read those, never reverse-engineer the minified bundle"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-09T15:02:44.160Z
---

**`@sellsuki-org/sellsuki-components` มีเอกสารของตัวเองใน `node_modules/@sellsuki-org/sellsuki-components/claude-templates/skills/`** — `use-ssk-components.md` (setup + framework quirks), `ssk-components-catalog.md` (prop/event/slot ครบทุก tag), `ssk-tokens.md`, `ssk-brand.md` · ติดตั้งเข้ารีโปด้วย `npx @sellsuki-org/sellsuki-components init-claude` (รี `update-claude` ทุกครั้งที่อัป package)

🔴 **`isCustomElement` ใน `vite.config.ts` เป็นข้อบังคับของ Vue 3 ไม่ใช่แค่ปิด warning** — เอกสาร DS ระบุว่า `:prop="value"` จะ **bind เป็น JS property** ก็ต่อเมื่อ Vue รู้ว่าเป็น custom element · ถ้าไม่ตั้ง Vue ส่งเป็น **attribute (string)** → component ที่รับ object/Date (เช่น `ssk-date-picker.value: Date`, `ssk-dropdown.options`) **พังเงียบ ๆ กดไม่ได้/ค่าไม่เข้า** ทั้งที่คอนโซลขึ้นแค่ "Failed to resolve component"
```ts
vue({ template: { compilerOptions: { isCustomElement: (tag) => tag.startsWith('ssk-') } } })
```

**บทเรียน (ผมพลาดเอง):** ตีความ "Failed to resolve component: ssk-*" ว่าเป็น cosmetic warning แล้วปล่อยไว้หลายรอบ ทั้งที่มันคือ**อาการของ property binding ที่พัง** · แล้วยังไปอ่าน bundle ที่ minify (`main-*.js`) เพื่อเดา API แทนที่จะเปิด catalog ที่มีอยู่แล้ว → สรุปผิดว่า `value` เป็น string (จริงคือ `Date`) แล้วแก้ผิดทางซ้ำ
→ **เจอ ssk-* component ที่ทำงานแปลก ให้เปิด `ssk-components-catalog.md` ก่อนเสมอ**

🔴 **popup ของ ssk-date-picker เป็น `position:absolute` จากใน shadow root — shadow root ไม่ใช่ containing block** ถ้า light DOM ไม่มี ancestor ที่ positioned ปฏิทินจะไปเกาะ page แล้วโผล่มุมซ้ายล่าง กดไม่ถึง · ต้องครอบด้วย `.calendar-container { position: relative !important }` (idiom ที่ `CampaignCondition.vue:379` / `CampaignRewards.vue:1451` ใช้อยู่แล้ว) · **เจอ ssk-* ที่ popup โผล่ผิดที่ = หา positioned ancestor ก่อนเสมอ**
- `displayok=""` → เลือกวันเฉย ๆ ไม่ commit ต้องกด "ตกลง" ถึงจะ fire `change` (ค่าเข้า `dp.value` เป็น Date จริง)
- input จริงซ้อน 2 ชั้น: `ssk-date-picker` → shadow `ssk-input` → shadow `input` · programmatic `.click()` ที่ input ไม่เปิด popup (ต้อง click จริง)
- event เป็น kebab-case (`ssk-change`) แต่บาง component ใช้ `change` เฉย ๆ — เช็ค catalog ราย component
- catalog อาจคลาดจาก build จริง (date-picker เขียนว่า emit `{valueFrom}` แต่ single picker ส่ง `{value}`) → รับทั้งสองแบบ

ดู [[project_oc2plus_apikey_local_run.md]]
