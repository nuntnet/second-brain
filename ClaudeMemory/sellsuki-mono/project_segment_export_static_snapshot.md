---
name: project_segment_export_static_snapshot
description: "BOLA static segment export must read members from the snapshot table, not re-evaluate the rule live"
metadata: 
  node_type: memory
  type: project
  originSessionId: 5cc01698-54e5-4bfb-b9e1-d4aba5865591
---

For BOLA (`backend/bola-backend`), exporting a **static** segment must pull members from its frozen snapshot (`segment_member_snapshots`), NOT by re-evaluating `seg.Rule` live. Dynamic segments evaluate the rule live.

**Why:** a static segment is a membership snapshot frozen at create/recalculate time. Re-evaluating the rule on export returns *current* matches, defeating the point of "static". Snapshot exists only for static FOLLOWER segments (see `buildStaticSnapshot`); dynamic + contact segments have no snapshot → live eval.

**How to apply:** in `runSegmentExportJob` ([src/use_case/segment.go](backend/bola-backend/src/use_case/segment.go)), branch on `seg.IsDynamic`/source type — for static follower segments fetch follower IDs from `segmentMemberSnapshotRepository` then load via `customerRepository.ListCustomersByIDs`. As of the BOLA-157 keyset fix the export still re-evaluates the rule for all segment types — this is a known gap to fix. Related: [[reference_bola_jira_project.md]]
