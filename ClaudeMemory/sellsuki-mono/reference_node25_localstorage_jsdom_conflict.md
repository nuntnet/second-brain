---
name: reference_node25_localstorage_jsdom_conflict
description: "this machine only has Node v25.8.0 installed (via brew, no nvm/fnm/volta) — its built-in global localStorage breaks jsdom's polyfill in vitest, even on pre-existing unrelated spec files"
metadata:
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-09-12T22:42:10.773Z
---

This machine has only Node v25.8.0 installed (`brew list --versions` → `node 25.8.0`, no `nvm`/`fnm`/`volta`/`asdf`/`mise` present). Several frontend repos here (e.g. `oc2plus-linecrm-frontend-backoffice`, `.nvmrc` → `v20.5.0`) pin an older Node for CI, but nothing enforces it locally.

Node 25's built-in global `localStorage` collides with jsdom's own `localStorage` polyfill under vitest (`environment: 'jsdom'`) — any spec file that calls `localStorage.setItem`/`.removeItem` throws `TypeError: ... is not a function`. Confirmed this is **not** a bug in new test code: the identical failure reproduces on a pre-existing, already-merged spec file (`src/views/Bola/ContactList.spec.ts`) that has nothing to do with whatever you're working on.

**How to apply:** if a vitest run fails on `localStorage.setItem is not a function` on this machine, don't debug the test logic first — check `node --version` (likely 25.x) and cross-check against an unrelated pre-existing spec that also touches `localStorage`. If it fails identically there too, it's this environment mismatch, not a real defect — note it and move on, or install the repo's pinned Node version (via `brew install node@20` or similar) for a clean local run if full verification is needed.

## 2026-09-13 — ตอนนี้ Node v26.8.2 และ **ข้อความ error เปลี่ยนไป**

อาการใหม่คือ **`Cannot read properties of undefined (reading 'removeItem')`**
(หรือ `'setItem'`) ไม่ใช่ `is not a function` แล้ว — ใครเสิร์ชด้วยข้อความเดิม
จะไม่เจอ แล้วไปไล่ debug ตรรกะเทสแทน

พิสูจน์ตรง ๆ ได้ด้วยคำสั่งเดียว ไม่ต้องเดา:
```
node -e "console.log(typeof globalThis.localStorage)"
# => undefined
# (node:…) ExperimentalWarning: localStorage is not available because --localstorage-file was not provided.
```
Node 26 นิยาม `globalThis.localStorage` เป็น **accessor (get/set)** ที่คืน
`undefined` เมื่อไม่ได้ส่ง `--localstorage-file` — แล้วมันไป **บัง polyfill ของ
jsdom** ใน vitest

**CI ไม่โดน**: `.gitlab-ci.yml` ของ member frontend ใช้ image `node:22`
(`UNIT_TEST_IMAGE_NAME`) ซึ่งไม่มี global ตัวนี้ → เทสเขียวบน CI

เจอครั้งล่าสุดที่ `src/react/i18n/memberLanguage.spec.tsx` (3 เคส, fail หมดทั้งไฟล์)
ส่วน `ProfilePageImpl.spec.tsx` ที่แตะ localStorage เหมือนกัน**ผ่าน** เพราะห่อ
`try { … } catch {}` ไว้ — อย่าเอาไฟล์ที่ผ่านมาสรุปว่าไม่ใช่ปัญหา env
