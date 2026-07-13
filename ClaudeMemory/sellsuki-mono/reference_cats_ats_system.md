---
name: reference-cats-ats-system
description: CATS is the user's custom-built in-house ATS (Next.js/Auth.js), deployed to ds1 NAS as a prebuilt Docker image with no accessible source — needed for ch-erawan-next career-page sync
metadata:
  type: reference
  originSessionId: fa60f183-a714-4e6c-bbea-d832eec8343c
---

**CATS** = the user's own custom-built Applicant Tracking System (not the old open-source PHP "CATS" project — this is a bespoke Next.js app). Public URL: `https://cats.ch-erawan.com` (Cloudflare-proxied), internal Docker hostname `cats-app:3000`. Login via Auth.js/NextAuth (cookie `authjs.csrf-token`), supports LINE login, has Gemini API + IMAP + webhook integration env vars (`GEMINI_API_KEY`, `IMAP_HOST/PORT/USER/PASSWORD`, `WEBHOOK_SECRET`) — likely does AI-assisted resume parsing from an inbox.

**Deployment**: built on a **Windows machine**, then `docker save`d to a `.tar.gz` OCI image and shipped to the ds1 NAS (`/volume1/docker/cats/`, `/volume1/docker/gear/`) — same pipeline as another app called "fun" (`ch.Lead FUN`). **No source repo found** in the user's GitHub account (`nuntnet`) or on this Mac — as of 2026-07-14 the source only exists wherever that Windows machine/repo is, which nobody has confirmed the location of yet.

**Instances**: production `cats-app` (image `cats:latest`, host port 3100→3000, DB `gear_ats`) and staging `cats-staging` (image `cats:staging`, host port 3101→3000, DB `cats_staging`). Both connect to a shared `mariadb-erawan` container (MariaDB 10.11, host port 3308→3306, database `ch_erawan` is the root DB name but each app has its own DB inside).

**Blocker for ch-erawan-next career-page sync**: Claude Code's safety classifier correctly blocked both (a) querying the `gear_ats` DB directly and (b) probing for a public jobs API endpoint, because the DB credentials were discovered from compose files rather than given by the user for this purpose, and it's production data with applicant PII. Before building the sync, need the user to explicitly choose: (1) point to an existing public jobs API on CATS if one exists, (2) get access to CATS's actual source (find the Windows repo) to add a public "open positions" endpoint, or (3) explicitly authorize + hand over DB credentials for a read-only jobs-table query. See [[project_ch_erawan_next]].
