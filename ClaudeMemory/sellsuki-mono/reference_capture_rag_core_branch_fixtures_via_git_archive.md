---
name: reference_capture_rag_core_branch_fixtures_via_git_archive
description: "Real rag-core wire fixtures from an unmerged branch without checking it out — git archive src+tests to scratch, run its test helpers/ASGI routes with backend/rag-core/.venv python"
metadata:
  node_type: memory
  type: reference
  originSessionId: cc30fdb9-527c-4eb2-92a5-823515ceebc9
  modified: 2026-11-20T11:29:51.653Z
---

To build a chat-core fixture from a rag-core branch that is not merged, without touching the main checkout:
1. `git -C backend/rag-core archive origin/<branch> src tests | tar -x -C $SCRATCH/rag`
2. Write the generator script in a different directory. From inside `$SCRATCH/rag`, run it as `PYTHONPATH=$SCRATCH/rag backend/rag-core/.venv/bin/python script.py`.
3. For review routes, reuse `tests.unit.interface.test_structured_review_route` (`build_app`, `configure`, `token`), `tests.unit.use_case.test_structured_review.setup(document=...)` and `test_source_read_purpose.request` to drive the real ASGI app. For use-case outputs, dump them with `pydantic.TypeAdapter(<dataclass>).dump_json`.

This was used for AI-500. The generators live in chat-core `scripts/structured-range-*.py`, and chat-core's older `scripts/structured-review-contract.py` follows the same pattern.

Also: inside a session worktree, the Write/Edit hook refuses any path outside that worktree. Create submodule worktrees inside it, e.g. with `git worktree move`.

Related: [[feedback_tests_derive_from_the_card_not_the_code]]
