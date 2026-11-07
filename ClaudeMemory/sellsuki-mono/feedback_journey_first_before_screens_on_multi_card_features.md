---
name: feedback-journey-first-before-screens-on-multi-card-features
description: "For a feature spread over several cards (e.g. how a pipeline case ends: AI-361/362/363/314/370), write the user journey and the real-service capability table first; don't fix screens one screenshot at a time"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 70429684-da98-4359-8018-c1ff7471777a
  modified: 2026-11-07T07:48:30.815Z
---

Retro with nunt, 2026-11-07 (AI-49 console reset, a day of rework on ends / invitation / question set / canned answers).

What actually drove the rework, in order:
1. Nobody owned the journey. "How does a case end" was split across AI-361 (terminals), AI-362 (system events), AI-363 (signals), AI-314 (goal/invitation), AI-370 (screens). Each card was detailed and testable on its own, but no card said what an admin does, start to finish. The model ("an end is an outcome; the main path is chat: invite → reply") only appeared after three rebuilds.
2. I built from my own belief of the service (briefs + mocks), not a probe of the real one, and "verified" one happy path. See [[feedback-probe-every-control-against-real-service-before-ui]].
3. Reactive loop: nunt points at one element, I fix that element, the next one breaks. Local fixes on an unsettled model.
4. Parallel agents on one surface coded against a contract I invented; their confident reports hid the gap. Also missed prior art (AI-398 already existed).
5. The approved prototype (v9 / g1) was not opened for the ends work; design was decided during implementation.
CPU load cost time (503s, retries) but did not cause the wrong designs.

**How to apply:** when a request touches a concept that spans cards, stop before code: (a) write the journey in plain steps per persona, (b) probe the real service and attach the capability table, (c) open the approved prototype and note where it disagrees, (d) get one nunt confirmation, then build — one agent per surface, never two on the same screen.
