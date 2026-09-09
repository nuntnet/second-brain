---
name: project_provider_create_lives_in_ccs1
description: Provider create UI belongs in system-management-frontend (CCS1, 5179), NOT provider-management-frontend (5178) — AI-21 AC1's text still names the wrong app
metadata:
  type: project
---

**Provider bootstrap (create) lives in `frontend/sellsuki-system-management-frontend`
(CCS1, port 5179), not `frontend/sellsuki-provider-management-frontend` (5178).**

Why: CCS checks `sellsuki.provider.create` in the **`SellsukiSystem`** Keto namespace.
A Provider Owner's role never carries it, so a create form on 5178 always 403'd. It was
a page that could never succeed, not a live authorization hole.

## Timeline (all 2026-08-02 unless noted)

| when | what |
|---|---|
| 16:22 | Jira comment 43134 claims MR !9 (provider-mgmt-FE) covers **AC1** — including a create screen |
| 23:42 | `292b9bb` **deletes** `/provider/create` from 5178 (`src/pages/Provider/Create.svelte`, −343 lines). MR !9 retitled *"create-provider UI removed, belongs in system-mgmt"* |
| 23:58 | `55c6120` **builds** `/provider` list + `/provider/new` create in system-mgmt-FE → **MR !76** |
| 08-28 | MR !9 merged into provider-mgmt main — **list only** |
| 09-09 | MR !76 finally merged, after retargeting `main`→`develop` (merge commit `bc3e616` on develop) |

So for ~5 weeks **no merged branch of any frontend had a provider-create UI** — provider
bootstrap was `POST /admin/provider` only. Verified across every frontend: 5178 main had
only `List.svelte` + GET; system-mgmt main *and* develop had zero provider files; the
`admin/provider` hits in company-management-frontend and sellsuki-invitation are in
`src/services/ccs.ts`, which is **generated OpenAPI types**, not UI.

## Card is still wrong

**AI-21 AC1** says *"ผ่านหน้าจอ provider management เดิม"* and Screen/Entry Point names
5178 with *"ไม่สร้างหน้าใหม่"*. Both were invalidated the same day comment 43134 claimed
AC1 was done, and never corrected. Reported as comment **44575** (2026-09-09), not edited
— see [[feedback_report_wrong_cards_dont_edit]].

## Repo gotcha

`sellsuki-system-management-frontend`'s active line is **`develop`**, not `main` — `main`
had not moved since 2026-08-02 while develop was 24 commits ahead. MRs in that repo are
inconsistent about target (!76 and !77 opened against main, !72 against develop). Check
the target before merging anything there. Same class as [[project_monorepo_mainline_is_not_main]].

Related: [[reference_backend_without_frontend_is_invisible]] — the backend endpoint existed
the whole time; only the screen was missing.
