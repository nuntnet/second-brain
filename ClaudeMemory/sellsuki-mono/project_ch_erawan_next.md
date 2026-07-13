---
name: project-ch-erawan-next
description: ch-erawan-next is the Ch.Erawan Group car dealer website (Next.js) — separate from sellsuki_mono/helio; career page job listings are hardcoded, wants CATS ATS sync
metadata:
  type: project
  originSessionId: fa60f183-a714-4e6c-bbea-d832eec8343c
---

`/Users/nunt/ch-erawan-next/` is the marketing/dealer website for **ช.เอราวัณ ออโต้ กรุ๊ป** (Ch.Erawan Auto Group), Nakhon Pathom — Mazda/Ford/Mitsubishi/GWM/Deepal/Kia across 7-9 branches. Next.js, deployed on Vercel (staging: `ch-erawanwebsite-git-staging-cherawan-next-website-s-projects.vercel.app`). GitHub repos: `nuntnet/ch-erawan-next-website` (public) and `nuntnet/ch-erawanwebsite` (private). This is a completely separate codebase/infra from the sellsuki_mono monorepo and the helio project — same user, unrelated stack.

**Career page** (`app/career/page.tsx`) currently has a fully hardcoded `jobListings` array (client component) — titles, branches, salary, requirements all static in source. As of 2026-07-14 the user wants this synced with **CATS**, their in-house Applicant Tracking System, so open positions on the site match what's actually open in the ATS. See [[reference_cats_ats_system]] for what CATS is and the blockers found so far (no accessible source, DB access needs explicit user go-ahead).

**Why:** avoid manually editing website code every time a job opens/closes — CATS should be the single source of truth for "currently hiring" positions.

**How to apply:** any career-page sync work must decide the integration point first (public jobs API on CATS vs. direct read of the `gear_ats` DB) — do not just start writing code against whichever is easiest; that decision was still open with the user as of this writing.
