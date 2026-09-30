---
name: reference_liff_single_endpoint_flow_in_link_not_endpoint
description: "A LIFF app has ONE endpoint URL; the rich-menu link carries flow=… . BOLA's LINE OA page used to show a per-flow endpoint per destination row — pasting …/liff/shell?flow=home fixed flow=home for every button (all three menu buttons opened Home)"
metadata:
  type: reference
---

Seen 2026-09-30 on dev: buttons `flow=home|pointclaim|reward` on one LIFF all opened Home; BOLA logs showed every `GET /v1/public/liff/destination` carrying `flow=home` although destinations for all three flows existed and were right.

Cause (inferred — the LINE console cannot be read from here): the LIFF endpoint held `flow=home`, and `LiffShellPage.readParam` took the first match. Correct endpoint has **no flow**: `https://<bola-web>/liff/shell?line_oa_id=<oa>`.

Fixed in bola-frontend !129: tapped `liff.state` wins, last direct value next, and the OA page now shows one flow-less endpoint (per-row ones removed). Diagnose fast by grepping BOLA request logs for `liff/destination` and reading the `flow` query values.

Also: `flow` names are free text keys of `liff_shell_destination (line_oa_id, flow)`; the standard OC2Plus preset uses `register`, `submit-claim`, `my-claims`, so a hand-made `home/pointclaim/reward` set is a separate convention. A missing row gives `liff_destination_not_found`.

Related: [[project_oc2plus_liff_shell_is_the_line_entry]], [[reference_bola_liff_shell_error_ladder]].
