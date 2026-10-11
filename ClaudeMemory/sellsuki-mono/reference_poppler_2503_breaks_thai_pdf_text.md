---
name: reference_poppler_2503_breaks_thai_pdf_text
description: "Thai PDF text looks fine locally and broken on staging because python:3.12-slim (Debian trixie) ships poppler 25.03, which splits Thai vowels/tone marks; Homebrew has 26.x. Same code. Fix = poppler >=26 in the worker image (data-pipeline !22)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-11T17:44:03.409Z
---

**Seen 2026-10-12 (AI-291):** the staging review screen showed imported Thai PDFs with marks floating off their consonants ("ปัจจุบน     ั", "ทัง  ้", "วา่"). The PO remembered "we fixed this on local". The fix they remembered (`0360c93` on unpushed `codex/AI-291-pdf-text-quality`) was already in main. The real difference was **poppler**: the worker image `python:3.12-slim` = trixie = poppler **25.03**, the Mac = Homebrew **26.04**.

Measured on "FWD Precious Protection_2025.pdf" with the same parser:
- 25.03: 36,847 chars and 156 detached marks. That length is exactly the canonical text stored on staging.
- 26.04 / 26.07: 31,864 chars and 47 detached marks.

**How to apply:**
- When "works on local, broken on staging" involves PDF text, compare `pdftotext -v` first. Reproduce in `docker run python:3.12-slim` with apt poppler-utils, mounting the repo.
- Fix shipped as data-pipeline MR !22: `scripts/install-poppler.sh` installs from Debian testing, pinned. It pulls glibc 2.43. A guard test fails on poppler < 26.
- Documents that were already imported keep the broken text until they are re-imported.

Related: [[reference_monorepo_docs_live_on_ai49_branch]].
