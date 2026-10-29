---
name: feedback-never-cut-ai-reply-gates-observe-only
description: "AI chat output gates must not replace or cut the model's reply to enforce business rules (D-060) — enforce in the prompt, log the finding; replaced replies sent nameless/irrelevant text on staging"
metadata:
  type: feedback
---

Nunt, 2026-10-29 (D-060, relayed through the AI-337 parent session): "ไม่ควรเอาข้อความตัดหาย ถ้าจะต้องตอบก็ตอบออกมาเลย".

The AI-337 playbook gate replaced any reply that quoted a premium before the step's facts were complete. On staging it fired on the very message that completed the step (one-turn checkpoint lag) and sent "…ขอข้อมูลเพิ่มเติมอีกเล็กน้อยนะคะ", which asked for nothing.

**Why:** a cut reply is a worse customer experience than an early figure, and a post-hoc rewrite cannot see enough context to ask the right thing.
**How to apply:** for business-rule gates (playbook step, quoting timing), put the rule in the prompt and record a guardrail finding plus a trace field. Do not swap the body. This does not cover safety/compliance gates that escalate (faq_only restricted topic, low confidence); those stay. Related: [[feedback-report-wrong-cards-dont-edit]].
