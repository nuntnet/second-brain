---
name: reference_robot_variables_are_case_insensitive_loop_var_overwrites_suite_var
description: Robot Framework variable names are case-insensitive - a FOR loop variable ${m} silently overwrites the suite variable ${M}; hit three times in the QA suites
metadata:
  type: reference
---

In the Robot suites under `testing/oc2plus-line-crm-automate-testing`, `${m}`, `${M}` and `${c}`/`${C}` are the SAME variable. A `FOR ${m} IN ...` loop (or a `${c}`/`${t}`/`${ua}` assignment) overwrites a suite variable like `${M}` / `${CO_A}` / `${UA}`, and nothing errors: the suite then seeds or calls the wrong member/company (seen as duplicate-key violations in setup, tokens that "belong" to the wrong member, a User-Agent header that became a response dict).

**How to apply:** give loop and assignment variables names that cannot collide with suite constants after lowercasing (`${mem}`, `${com}`, `${tok}`, `${bind_co}`, `${acc_member}`, `${r_ua}`); when a fixture fails in a way that "cannot happen", check this first. Related traps seen with the same suites: OTP/replay/rate-limit counters in Redis are under `auth_guard:*` (a fixture purging `line_login:*` clears nothing), and `rtk grep -E "--- FAIL"` treats the pattern as an option (use `grep -e`). See [[reference_local_qa_run_facts]].
