---
name: console-v9-frontend-is-a-mess-reset-before-more-screens
description: "2026-11-05 nunt judged the deployed v9/M5 console a mess — mocks not good, old+new mixed, harder to use than before, AI slop; fix everything before building more screens"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 28036f73-9028-4b3a-98a8-7ab8e1f2c879
  modified: 2026-11-05T13:14:38.057Z
---

2026-11-05, after seeing M5 G1–G5 live on staging and the AI-369 mocks, nunt said:
1. the mocks themselves are not good-looking;
2. old and new screens are mixed together at random;
3. the console is cluttered and harder to use than before;
4. lots of AI slop; everything needs fixing.

**Why:** screens were built card by card by agents from a behaviour prototype (v9) plus agent-made mocks, each card adding its own panels, and nobody judged the whole console as one product before shipping. Green tests and AC tables did not catch it.

nunt, same day: "V9 มันก็เป็นแค่ mock ที่ยังไม่ใช่ ui จริงๆ การหยิบมาใช้ต้องหยิบเฉพาะของดีๆนะ ไม่ได้ทำให้เหมือน v9". v9 is an idea source, not a spec. Cards and ACs that say "as in v9" / copy every v9 panel are part of the cause. Take only the parts that make the job easier and drop the rest. Every v9 element has to earn its place.

**How to apply:** do not add more console screens (AI-375 G6 etc.) on top of the current state. First audit the whole console as a user would (route inventory, old vs new, duplicates, clutter, slop patterns), agree an information architecture and a visual direction with nunt, then rebuild. A mock or screen is not "done" until nunt has looked at it beside the real page. Related: [[console-ux-story-presets-icons-generic]], [[design-handoff-full-fidelity]].
