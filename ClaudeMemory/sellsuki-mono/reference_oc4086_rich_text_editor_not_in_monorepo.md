---
name: reference_oc4086_rich_text_editor_not_in_monorepo
description: OC-4086 (Text Editor + sanitizer สำหรับเอกสาร Consent) สถานะ Done แต่โค้ดไม่มีใน monorepo เลย — การ์ดอื่นที่สั่งให้ reuse จะหาไม่เจอ อย่าเสียเวลาไล่หา
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-13T15:25:40.117Z
---

OC-4356 (News CMS) เขียนไว้ว่า "reuse rich text editor + sanitizer whitelist ของ OC-4086" ·
OC-4086 สถานะ **Done** · **แต่ของไม่มีอยู่จริงในรีโปไหนเลย** (ตรวจ 2026-09-13):

- ไม่มี `tiptap` / `quill` / `ckeditor` / `toast-ui` / `editorjs` ใน `package.json` ของ frontend ใดใน monorepo
- `git log --all --grep=4086` ใน frontend-backoffice, backoffice-api, company-management-frontend,
  service-consent = ไม่มี commit
- backoffice-api ไม่เคยมี `bluemonday` มาก่อน OC-4356

→ agent เสียเวลาไล่หาก่อนจะยอมสร้างใหม่ · ที่ลงจริงใน OC-4356 คือของใหม่ทั้งคู่:
FE เป็น `contenteditable` + `execCommand` ขั้นต่ำ (`components/News/NewsRichTextEditor.vue`),
BE เป็น bluemonday policy ตาม whitelist ในการ์ด (`use_case/news_sanitizer.go`:
`h1,h2,h3,p,strong,em,u,ul,ol,li,a[rel=noopener],hr` + `text-align`, href เฉพาะ http(s))

**Why:** เป็นตัวอย่างสดของ rule `design-doc-authority.md` ข้อ "Never assume a Jira status reflects reality"
— Done ไม่ได้แปลว่าโค้ดอยู่ในที่ที่การ์ดอื่นจะ import ได้

**How to apply:** การ์ดไหนสั่งให้ reuse ของ OC-4086 → สร้างใหม่ตาม whitelist ในการ์ด แล้ว flag เป็น deviation
ถ้าวันหนึ่งเจอของจริง (อาจอยู่นอก monorepo) ค่อยรวมกัน — อย่าไล่ grep ซ้ำ

เชื่อม [[project_oc4356_news_cms_dev_blockers]]
