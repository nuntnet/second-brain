---
name: chat-core-guard-bans-stage-word-outside-lead-packages
description: chat-core src/guard lead_boundary_test fails on the word "stage" in team/conversation packages; put stage-named strings in an exempt entity package
metadata:
  type: reference
---
`src/guard/lead_boundary_test.go` (TestNoLeadStageVocabularyInDataModel) fails if the word "stage"/"Stage" appears — even in a comment, error code or message — in non-Lead-Engine packages such as `use_case/team` and `route/team`. Hit on AI-399 adding `stage_entry_not_supported`.

**How to apply:** put the code/message/check in a package that already owns the vocabulary (e.g. `entity/alert_recipient`, which defines FamilyStageEntry) and call it from team code via neutral names (`ar.CheckWritable`, `ar.IsReserved`). Also: chat-core cmd uses caarlos0/env **v10**.
