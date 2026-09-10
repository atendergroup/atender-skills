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

`voiceLanguageVoices` takes ElevenLabs voice ids. `ttsVoice` and `voiceLanguages`
in the voice settings are Amazon Polly names, used by IVR flow nodes. The two
lists are not interchangeable, and a Polly name in the stack map breaks the call.

No tool returns the ElevenLabs catalogue today. Take ids from
`https://www.atender.com/docs`, or ask the customer for the ids they were given.
Do not guess an id.

## Choosing the transfer target

Exactly one of these resolves, in this order:

1. `voiceFallbackQueueId` — one fixed queue. Simplest, and the one to use when there is one place callers go.
2. `voiceHandoverAiTeam: true` — the assistant picks among teams whose queues carry a `description`. Every candidate queue needs a description or it is invisible to the assistant.
3. The queues of `handoverTeamId` — a team with several queues needs a `description` on each.

With none of them resolvable the stack has no transfer at all. Then the business
rules must say the assistant cannot transfer, so it does not promise one.

## Fallback

`voiceFallbackAction` defaults to `hangup`. Set it to `queue` (with
`voiceFallbackQueueId`) or `voicemail` on purpose. A queue with nobody online
holds the caller with no limit — pair it with an `is-open` node and a call back
in the IVR flow.

## The business rules block (VO-08)

Write these into the voice stack's business rules, and nowhere else:

- Say one thing at a time, at most two short sentences.
- Answer a question that comes with a yes.
- For an identifier, list what the caller has and let them choose by number.
- Ask before you transfer.
- Before ending, ask once if there is anything else.
- Never say a filler such as "one moment".
