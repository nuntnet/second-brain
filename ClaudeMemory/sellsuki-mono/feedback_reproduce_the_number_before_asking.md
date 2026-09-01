---
name: feedback_reproduce_the_number_before_asking
description: Never ask the user to decide based on a number read out of a log — run the command yourself first, or the decision is made on an artifact
metadata:
  type: feedback
---

I reported "CI says coverage is 46.5%, threshold is 70" straight from a job log and
asked whether to lower the threshold. The user answered "ตั้ง 45". One command later
`make check-coverage` returned **72.1%** — the 46.5 was an artifact of a CI job with
no database (see [[reference_coverage_gate_on_dbless_job]]).

Halting and re-presenting was right: the user replied "ทำ ก เลย" and took the real
fix instead. But the whole exchange existed only because I had not run the command.

**Why:** a number quoted out of a log is not a measurement, it is a claim about an
environment I did not inspect. Handing it to the user converts my unverified reading
into their decision, and "45" would have silently retired a working gate.

**How to apply:** before asking the user to choose a threshold, limit, timeout, or any
figure, reproduce it locally and say which environment produced which value. If the
two disagree, that gap IS the finding — lead with it rather than with the choice. And
when a decision the user already made rests on a number that later proves wrong, stop
and say so before executing; do not carry out an instruction whose premise I supplied
and then broke. Same discipline as [[feedback_ground_claims_file_line]] and
[[feedback_verify_absence_claims]], applied to numbers.
