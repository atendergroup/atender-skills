---
name: atender-ivr
description: Set up or audit an Atender phone menu over the Atender MCP server — call queues, the phone number, the IVR flow and its graph of nodes and edges, retries and timeouts, opening hours, publishing the flow and binding it to the number, then testing every branch. Use this skill whenever the user asks what a caller hears before reaching a person; when they say "phone menu", "IVR", "press 1 for", "call queue", "callers hear the number is unavailable", "route calls by language", "out of hours message on the phone", "provision a number", or "publish the flow"; and whenever an Atender MCP connection is present and the work is about the path a call takes. What the AI assistant says on the call is the atender-agent-stack-voice skill.
---

# Atender phone menus (IVR)

## When to use

Use this when a caller has to get somewhere — a team's queue, a call back, the AI
assistant, voicemail or a message — in as few keys as possible. It owns call
queues, the phone number, the flow graph, opening hours on the phone, publishing
and binding, and the test script.

What the AI assistant says once a call reaches it belongs to
`atender-agent-stack-voice`.

## Before you start

Read `../_shared/atender-setup-basics.md` first — connection, precedence, the
order of areas, the preflight, pacing, the report format and the groundwork
areas. Then call `describe_configuration_model`; it has no voice or IVR section
today, so this skill wins on order and on how to test.

The order is: teams, then call queues, then the number, then the flow, then the
definition, then the bind, then publish. Without a queue the flow write answers
201 and the number never syncs — the only symptom is `provider.configured:
false`.

Read `get_ivr_flows` for each existing flow first and tell the customer what a
caller hears today.

## Needs from the customer

- The greeting and each choice, in each language, and the key for each choice.
- Where each key goes: a team's queue, a call back, the Agent Stack, voicemail or a message.
- The language key, if there is one.
- The opening-hours rule, and what a caller gets out of hours.
- Whether calls are recorded. This is a legal decision for the customer.
- Whether a number is assigned to this workspace. If not, they request one in the app. That costs money.

## Checklist

- **IV-01** Queues first, always: `create_voice_call_queues` **before** `create_ivr_flows` — a flow saves with 201 without one and the number then never syncs. One queue per team, each with a `description` (the assistant routes only to described queues). `holdAudioType` and `reservationTimeout` as agreed, and **`callbackEnabled` with a low trigger position**: a queue holds a caller with no ceiling and there is no maximum hold time, so the call back is the only way out.
- **IV-02** Number: `phoneNumber` starts with `+`, `capabilities.voice` is true, and there is one row per number in `list_voice_phone_numbers`. If not, stop.
- **IV-03** `create_ivr_flows`: `ttsLanguage` (BCP-47) and `ttsVoice` (a Polly name). No `welcomeMessage`.
- **IV-04** Definition: `update_ivr_flows_definition` with the whole graph, written before the bind. The node types, their required data and the edge handles are in `references/flow-graph.md`.
- **IV-05** Check the graph yourself: the validator checks none of it. Walk the list in `references/flow-graph.md`, and confirm the node and edge counts match what you sent.
- **IV-06** Bind: `update_voice_phone_numbers` with `ivrFlowId`, `mainAgentId` and `defaultAgentLanguage` in one call. If `provider.configured` is false, call `create_voice_phone_numbers_provision` once. On a 409, stop and tell the customer.
- **IV-07** Publish: after **every** `update_ivr_flows_definition`, call `publish_ivr_flows` with a comment and then check `provisioningError` is null. An unpublished definition changes nothing a caller hears, and a publish that left `provisioningError` set has not reached the carrier.

## Rules

- Check what a number is before you diagnose anything after it: one row, `+` prefix, `capabilities.voice` true.
- A write is live on a bound number. Write the definition before the bind.
- After a change to a number, a stack or a flow, the first call more than 30 seconds later reads the change.
- The greeting is at most two sentences: who we are, what the line is for, then the first choice.
- A test call is a real call that costs money. Get a yes before each one, and do not score the first call.

## Verify

- `get_ivr_flows`: the node and edge counts match what you sent, and `publishedRevision` is set.
- `list_voice_phone_numbers`: `ivrFlowId`, `configured` true, a recent `syncedAt`, `provisioningError` null.
- `list_voice_call_queues`: every queue that the flow or the voice stack routes to has a `description`.
- One call per branch, from the test script in `references/flow-graph.md`.

## What must be done in the app

Requesting a phone number, which costs money. Deciding whether calls are
recorded, which is the customer's legal call. Turning the AI voice feature on,
which only Atender can do. Tell the customer which are waiting on them.
