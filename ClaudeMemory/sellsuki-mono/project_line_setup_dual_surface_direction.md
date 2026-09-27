---
name: project_line_setup_dual_surface_direction
description: "PO direction 2026-09-27 for LINE setup: 1 company = 1 BOLA workspace; LINE OA + LIFF + destinations configurable from BOLA AND OC2Plus but stored only in BOLA (OC2Plus proxies); entry = Settings→LINE OA or Readiness row; multi-OA per company with one CRM; N companies→1 workspace parked"
metadata:
  type: project
---

Stated by the user 2026-09-27 while scoping OC-4592 / BOLA-333:

- **1 company : 1 BOLA workspace** — more is "เละ"; if the link breaks OC2Plus loses all LINE. BOLA already enforces it (`workspace.go:216` refuses re-bind and unbind), CRM enforces it (`ON CONFLICT (company_id)`).
- **Configure LINE OA settings and LIFF destinations from both BOLA and OC2Plus, but "จริงๆยิงเข้า bola backend"** — BOLA is the single store; OC2Plus is another entry point, never a copy.
- **Entry points:** sidebar "การตั้งค่า → LINE OA" and the Readiness LINE row → one page that shows binding status, lets you connect manually, and warns what breaks if unbound.
- **Minimum setup** is the goal ("แทบไม่ต้องตั้งค่าอะไรเลยยิ่งดี") — but the LIFF step is unavoidable: merchant creates their own LIFF (see [[reference_shared_liff_id_is_pnp_not_oc2plus]]).
- **Many LINE OAs per company, one member base** — wanted (departments/teams); data model already supports it (`member_bola_follower_link (member_id, line_oa_id)`).
- **N companies sharing 1 workspace** — asked about; I recommended parking it (tenant scoping reads OAs by workspace id in 3 places + BOLA refuses by design). Not decided by the user.

**Open decision (not answered yet):** whether OC2Plus writes to BOLA with the acting user's identity (BOLA checks workspace admin — my assumption in the cards) or OC2Plus company admins get BOLA rights automatically. Ties to OC-4591.

Related: [[project_oc_bola_domain_boundary]] · [[reference_company_workspace_link_lives_in_three_stores]] · [[reference_bola_sweep_binds_without_an_owner]]
