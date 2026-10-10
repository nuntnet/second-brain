---
title: Helio Control Tower
project: Ch-Erawan
owner: Botty
status: step-1-done
created: 2026-10-10
updated: 2026-10-10
tags: [helio, ch-erawan, control-tower, sps, mpr]
---

# Helio Control Tower

## Goal
One daily view of every Ch. Erawan branch (sales bookings + service RO / labour / parts) vs the MPR monthly targets, so lagging centers are flagged early in the month rather than at month end.

**Owner:** Botty · **Approved by:** Nunt (2026-10-10)

## Steps

| # | Step | Status | Date |
|---|------|--------|------|
| 1 | Daily SPS → Helio pipeline (bookings + service from SPS into helio-pg; MPR Oct targets loaded; MTD-vs-target view) | ✅ Done | 2026-10-10 |
| 2 | LINE image reading: daily center reports in the director group → `daily_service_kpi` / `mtd_reported` (vision + sanity checks, human review on flags) | ⏭ Next | — |
| 3 | MPR targets for every month + 08:00 morning summary from `v_mtd_vs_target` | Planned (Oct targets loaded) | — |
| 4 | Showroom milestones: GAC / LEPAS / Geely | Planned | — |

## Components created (step 1)

**helio-pg (NAS, new tables only, nothing existing touched)**
- `daily_booking_kpi`: bookings per day (`boo_create`) per center. gross, cancelled (boo_st_id 6), net, by_status.
- `daily_service_kpi`: RO closed + labour/parts per day of `job_complete`. Payer split: c (customer), internal (เคลมใน), w (manufacturer warranty), ins (insurance). Sources: `sps`, `line_manual` (and later `line_vision`).
- `monthly_target`: Oct 2026 targets. MPR is primary. Mazda NKP service comes from the LINE report (source `line_report_2026-10-10`). Mitsu 320k labour is stored as secondary.
- `mtd_reported`: MTD figures stated in center reports (LINE 10/10: Mitsu, KIA, Mazda NKP).
- View `v_mtd_vs_target`: MTD actual (SPS daily sum first, otherwise the latest reported MTD) vs primary target, % of target, % of month elapsed, status on_pace / watch / behind / no_data / no_target.
- Center codes: MZ-NKP, MZ-SLY, DP-SLY, FD-NKP, MT-NKP, GW-NKP, KI-NKP, LP-NKP, GL-RCD.

**n8n3**
- Workflow `Helio - SPS daily ingest` (id `GiZgVmBbMYPNzL3e`, active): Webhook POST with header auth → validate → upsert. Credential `Helio SPS ingest secret (header)` (id `A3S5eFUTKf3nZ77h`). Postgres via `helio-pg (LINE capture)` (id `hjiLFiqlLgKoBDLZ`).
- Existing: `Helio - LINE group capture (draft)` (id `tn9BWEIzEknVHbXJ`) → `line_messages` / `line_media`.

**Mac (MacBook Air)**
- `~/.helio/sps_daily.py` + `sps_daily.sh`: stdlib Python. Read-only SPS queries (session READ ONLY, max_execution_time 5000, PK/ID-window only), last 3 days ending yesterday, POSTs to the webhook with system curl. If the VPN/SPS host is unreachable it logs SKIP and exits 0.
- `~/Library/LaunchAgents/com.helio.spsdaily.plist`: daily at 05:00 (runs on wake if the Mac was asleep).
- Logs: `~/.helio/logs/sps_daily.log` (+ launchd.out/err.log).
- Secrets live only in 600 files under `~/.helio/` (not in this note).
- Manual backfill: `~/.helio/sps_daily.sh --from 2026-10-01 --to 2026-10-10`

## Verification (2026-10-10)
Backfill Oct 1–10: 90 booking rows (gross 109, cancelled 48, net 61) and 30 service rows (RO 312, labour 382,110). This matches the workbook `daily_vs_mpr_2026-10.xlsx` exactly. Re-runs are idempotent, and the launchd test run exited 0.

MTD vs target as of 10/10 (pace 32.3%):
- Mazda NKP: RO 133/580 (23%) · labour 182,180/500k (36%)
- Mazda SLY: RO 110/520 (21%) · labour 140,080/676k (21%), **behind**
- Deepal SLY: RO 69/104 · labour 59,850/67.6k (89%)
- Mitsu NKP (LINE): labour 83,610/340k (25%) · B&P 3,698/400k (0.9%), **behind**
- KIA NKP (LINE): labour 109,106/106,685 (102%). Target looks too low.
- Ford / GWM: no daily data.
- Bookings net 53/190.8 (28%, excl. Lepas). GWM 11/63 and Mazda 4/21.4 are behind.

## Open issues
- Hermes `N8N_MCP_TOKEN` returns 401. Needs a new token.
- Old Gemini API key still needs to be revoked.
- Rollback JSON files left on the NAS. Review and clean up.
- No daily service data for Ford, GWM. Mitsu only 6–10/10, KIA only 10/10 (until step 2).
- SPS body & paint (`adam_jobbp`) not ingested yet.
- Data mismatches to raise with centers / MPR owner:
  1. Mazda NKP has no MPR target Sep–Dec. The LINE report uses RO 580 / labour 500k / parts 1.875M.
  2. Mitsu SV labour target: MPR 340k vs LINE 320k.
  3. KIA has an MPR target but its LINE report has none. KIA B&P sheet is a blank template.
  4. Mazda Salaya sales targets are identical to NKP (copied?).
  5. GWM labour 150,334 vs 130,334 (sheet "รอตรวจSV").
  6. Deepal C+W 345,000 ≠ total 358,800. Oct/Nov actuals pre-filled.
  7. Ford B&P sheet is titled "KIA 2025".
  8. KIA counter target 300k includes film (parts only = 1,500).
  9. Mitsu LINE report: weekday names 1 day off, a 11/10 row with data, B&P RO total shows 0.
  10. Mazda NKP LINE header (RO 21 / MTD 123 / labour 161,400) ≠ its own daily table (22 / 133 / 168,900). SPS matches the table. On 3/10 SPS has an extra internal-claim 500 labour + 4,700 parts.
  11. Booking cancellation rate 44% (Ford 8/15, KIA 7/11). Check for duplicate bookings.
  12. Two bookings on 8/10 have no branch_id (GWM brand). Assigned to GW-NKP.
