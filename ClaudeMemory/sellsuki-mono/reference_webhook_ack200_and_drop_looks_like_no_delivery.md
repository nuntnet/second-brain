---
name: reference_webhook_ack200_and_drop_looks_like_no_delivery
description: messaging-backend read only entry[0].messaging[0], so a Meta batch like [read, message] was dropped whole with a 200 — indistinguishable from "Facebook never delivered", and it cost a full day of chasing Meta-side hypotheses
metadata:
  type: reference
---

`parseFirstFBInboundEvent` read **entry[0].messaging[0]** and refused the POST
if that item was not a message. Meta batches: `[read, message]` and
`[delivery, message]` are routine. Every such POST was thrown away **and
answered 200**, so Meta never retried and the customer's message was gone.

Fixed 2026-09-17 (messaging-backend MR !58, `27c41e0`): scan every entry and
every messaging item, take the first that carries a message, and take the page
id from the entry it was found in — routing keys off that, so entry[0]'s id
would deliver into another workspace's inbox.

Still one message per POST. A batch with two real messages processes the first
and drops the second, because dedupe, the replay claim and the forward are all
built on "one request, one event".

## Why it burned a day

The symptom is **identical to the third party not sending anything**:

- binding `healthy`, page subscribed in Meta's own console, token valid,
  signature passing
- we answer **200**, so Meta shows no delivery errors and never retries
- the rejection metric is app-wide, with no page label
- the log line said only `chat: invalid webhook event payload`

So a day went into Meta-side theories — page subscription, Development vs Live,
app secret, OAuth, Handover Protocol — all of them wrong, all consistent with
the evidence available.

**What broke it open was making the rejection say what it dropped**
(`8d24be4`: stage, page id, and the top-level keys of entry[0]). Deployed
22:56, caught the real payload at 23:14, fixed by 23:29, message flowing at
23:32. The instrumentation was worth more than every hypothesis.

General form: **an endpoint that swallows input and returns success is
unfalsifiable from outside.** When something "never arrives", instrument your
own refusal path before theorising about the sender.

Related: [[reference_procfile_messaging_disables_chat_module]] (the same shape:
our own 404 looked like "they never sent it").
