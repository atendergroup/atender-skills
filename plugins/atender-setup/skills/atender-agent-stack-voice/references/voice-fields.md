# Voice stack fields

## Language codes, per field

| Field | Form |
| --- | --- |
| `voiceLanguageVoices` keys | primary subtag (`no`, `sv`, `da`, `en`, `de`, `fr`) |
| `voiceWelcomeGreetings` keys | the same keys as `voiceLanguageVoices` |
| `defaultAgentLanguage` on a phone number | primary subtag |
| `send-to-ai-agent.languageCode` in an IVR flow | primary subtag (`no`, `en`) |
| `language` on an IVR `say-message` or `gather-input` node | BCP-47 (`nb-NO`, `en-GB`) |
| `ttsLanguage` on an IVR flow | BCP-47 |

## Voice ids

**Create the stack without `voiceLanguageVoices`.** The server fills the map with
valid ids from its own catalogue for the languages the stack answers in. Read the
result back with `get_agent_stacks`, tell the customer which voice each language
got, and change an entry only if they want a different one.

No tool lists the catalogue, so there is no way to check an id you typed by hand.
Do not ask the customer for ids, do not copy them from a docs page, and never
guess one — a wrong id makes every call in that language fail.

`ttsVoice` and `voiceLanguages` in `list_voice_settings` are Amazon Polly names,
used by the phone menu's own nodes. The two lists are not interchangeable, and a
Polly name in the stack map breaks the call.

## Choosing the transfer target

Exactly one of these resolves, in this order:

1. `voiceFallbackQueueId` — one fixed queue. Simplest, and the one to use when there is one place callers go.
2. `voiceHandoverAiTeam: true` — the assistant picks among teams whose queues carry a `description`. Every candidate queue needs a description or it is invisible to the assistant.
3. The queues of `handoverTeamId` — a team with several queues needs a `description` on each.

With none of them resolvable the stack has no transfer at all. Then the business
rules must say the assistant cannot transfer, so it does not promise one.

## Fallback

`voiceFallbackAction` defaults to `hangup`. Set it to `queue` (with
`voiceFallbackQueueId`) or `voicemail` on purpose.

A queue holds a caller with no ceiling — there is no maximum hold time to set.
Two things keep a caller out of an endless hold, and you need both: set
`callbackEnabled` on the queue with a low trigger position, so a caller is
offered a call back rather than a wait; and put an `is-open` node before every
send-to-queue in the flow, so a closed team is never handed a call at all.

## The business rules block (VO-08)

Write these into the voice stack's business rules, and nowhere else:

- Say one thing at a time, at most two short sentences.
- Answer a question that comes with a yes.
- For an identifier, list what the caller has and let them choose by number.
- Ask before you transfer. `handoverAskConfirmation` is not read on a call, so this rule is the only ask-before-transfer there is today.
- Before ending, ask once if there is anything else.
- Never say a filler such as "one moment".
