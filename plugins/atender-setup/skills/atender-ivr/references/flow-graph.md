# The IVR flow graph

`update_ivr_flows_definition` takes the whole graph in one write. Write it before
you bind the flow to a number, because a write is live on a bound number.

The queues come first. `create_voice_call_queues` before `create_ivr_flows`:
without a queue the flow write answers 201 and the number never syncs.

After every `update_ivr_flows_definition`, call `publish_ivr_flows` and check
`provisioningError` is null. An unpublished definition changes nothing a caller
hears.

## Node types

| Node | Required data | Edges it must have |
| --- | --- | --- |
| `incoming-call` | none | one out. Exactly one of these per flow |
| `gather-input` | `prompt`, `retryPrompt` (silence), `invalidPrompt` (wrong key), `maxRetries` (the number of asks, default 3), `timeout` (seconds, 1 or more), `language` (BCP-47), `voice` | one `digit-N` per key offered, plus a `timeout` edge |
| `language-selection` | `languages[{languageCode, languageName, dtmfKey}]`, `prompts` for each code | one `lang-<key>` per key |
| `say-message` | the message, `language` (BCP-47), `voice` | one out |
| `is-open` | `openingHoursRuleId` | `true` and `false` |
| `send-to-queue` | the queue id | none |
| `send-to-ai-agent` | `agentId`, `languageCode` (primary subtag: `no`, `en`) | none |
| voicemail | as the live schema says | none |

Read the live tool schema for the exact field names and for any node type not
listed here. The schema wins.

## Rules for the graph

- Exactly one `incoming-call`.
- An `is-open` node comes before **every** `send-to-queue`. A queue holds a caller with no ceiling, and there is no maximum hold time to set — pair the `is-open` with `callbackEnabled` and a low trigger position on the queue itself (IV-01).
- Every key you name in a prompt has a matching `digit-N` edge. A key with no edge is a dead press. This one you read, you do not compute: a `gather-input` declares no list of keys, so there is no list to check the edges against.
- Every keyed `gather-input` has a `timeout` edge, for the caller who presses nothing.
- No self edges. A wrong key already counts against `maxRetries`; a self edge loops the caller with no way out.
- Never add a `language-selection` node only to change the voice. `say-message` and `gather-input` each carry their own `language` and `voice`.
- The greeting is at most two sentences: who we are, what the line is for, then the first choice.

## Language codes, per field

| Field | Form |
| --- | --- |
| `ttsLanguage` on the flow | BCP-47 (`nb-NO`) |
| `language` on `say-message` and `gather-input` | BCP-47 |
| `languageCode` on `send-to-ai-agent` | primary subtag (`no`, `en`) |
| `defaultAgentLanguage` on the number | primary subtag |
| `languageCode` in `language-selection.languages` | as the live schema says; check it |

`ttsVoice` on the flow and `voice` on a node are Amazon Polly names. The Agent
Stack's `voiceLanguageVoices` takes ElevenLabs ids. They are not interchangeable.

## Retries and timeouts

- `maxRetries` is the number of asks, not the number of extra asks. Default 3.
- `timeout` is in seconds and must be 1 or more.
- `retryPrompt` plays when the caller says and presses nothing.
- `invalidPrompt` plays on a key that is not offered.
- After `maxRetries`, the flow follows the `timeout` edge. Send it somewhere a person can help, not back to the menu.

<!-- site:skip -->
## Checking the graph yourself (IV-05)

The validator does not check any of this. Before you publish, walk the graph and
confirm:

1. Every keyed `gather-input` has a `timeout` edge.
2. No edge points at its own source node.
3. Every `send-to-queue` is preceded by an `is-open`.
4. The node and edge counts in `get_ivr_flows` match what you sent.
5. Every queue the flow sends to has `callbackEnabled` with a low trigger position.

There is no "one edge per offered key" check to run mechanically: a
`gather-input` declares no list of keys. Read each prompt and confirm by eye that
every key it names has a `digit-N` edge.

## The test script

One call per branch, and read each one back:

- Each key that is offered, one call each. Check where the call lands.
- One wrong key. The `invalidPrompt` plays and the retry count moves.
- One call that presses nothing at all. The `timeout` edge is followed after `maxRetries`.
- One call per language, if there is a language key. Check the language and the voice of every step, not only the first.
- One call outside opening hours. The `false` branch of `is-open` runs.
- One call that reaches `send-to-ai-agent`, to check the handover from the menu into the assistant.

Then `publish_ivr_flows`, and `get_ivr_flows` and `list_voice_phone_numbers` for
the counts, `publishedRevision`, `configured`, `syncedAt` and
`provisioningError` (which must be null). Wait more
than 30 seconds after a change before the first call that should read it.
