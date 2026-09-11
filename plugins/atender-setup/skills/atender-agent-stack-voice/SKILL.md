---
name: atender-agent-stack-voice
description: Set up or audit an Atender voice Agent Stack over the Atender MCP server — the voice stack type, a voice per language, the greeting, pace, transfer queues and the fallback when the assistant cannot help, plus the parts of handover and customer verification that behave differently on a call. Use this skill whenever the user asks to build, configure, fix or review an Atender AI assistant that answers the phone; when they say "voice agent", "the AI on our phone line", "it answers in the wrong voice", "callers get cut off", "transfer to a queue", "what do callers hear"; and whenever an Atender MCP connection is present and the work is about a spoken conversation. The phone number, call queues and the key menu are the atender-ivr skill; text channels are atender-agent-stack-text.
---

# Atender Agent Stacks: voice

## When to use

Use this when the customer wants an Atender AI assistant to answer calls, or
wants to change, review or diagnose one. It owns the voice stack, its voices,
what the caller hears first, how fast it speaks, where it transfers, and what it
does when it cannot help. It also owns the parts of handover and verification
that behave differently on a call.

Phone numbers, call queues and the key menu belong to `atender-ivr`. Everything a
text channel does belongs to `atender-agent-stack-text`.

## Before you start

Read `../_shared/atender-setup-basics.md` first — connection, precedence, the
order of areas, the preflight, pacing, the report format and the groundwork
areas. Then call `describe_configuration_model`. The live tool schemas win for
field names and shapes; this skill wins over `describe_configuration_model` for
the order of work and for how to test.

Teams and call queues come before the stack can transfer anywhere.

## Needs from the customer (or from `atender-profile.md`)

- Which languages the line answers in. Not the voice ids — the server picks those (VO-03); ask only whether the voice it picked is the one they want.
- The first sentence a caller hears, for each language.
- Pace: fast, normal or patient.
- What happens when the assistant cannot help: a queue, voicemail or hang up.
- Whether callers must reach anything that needs a verified customer.

## Checklist

- **VO-01** AI voice feature = on. A voice create that answers 403 means it is off. Only Atender can turn it on. Stop and tell the customer.
- **VO-02** One voice stack (`type: "voice"`), separate from the text stacks.
- **VO-03** **Create the voice stack without `voiceLanguageVoices`.** The server fills the map with valid ids from its own catalogue. Read them back with `get_agent_stacks` and change one only if the customer wants a different voice. Do not ask the customer for ids, and do not take them from a docs page — you have no way to check an id you typed, and a wrong id breaks every call. Never copy `ttsVoice` or `voiceLanguages` from `list_voice_settings`: those are Amazon Polly names, for the phone menu's own nodes only.
- **VO-04** `voiceWelcomeGreetings` = one sentence for each key in VO-03: the company name, then what the line is for. Leave `voiceGreetingPrimaryLanguage` out. Nothing reads it.
- **VO-05** `voicePace` = the customer's answer (default `balanced`).
- **VO-06** Transfer target = `voiceFallbackQueueId` (one fixed queue), or `voiceHandoverAiTeam: true` (the AI picks among described teams), or the queues of `handoverTeamId`. A team with several queues needs a `description` on each queue. With no resolvable queue the stack has no transfer, and it must say so.
- **VO-07** `voiceFallbackAction` = `queue` (with `voiceFallbackQueueId`) or `voicemail`, not the default `hangup`. A queue holds a caller with no ceiling, so set `callbackEnabled` on that queue with a low trigger position, and put an `is-open` node before every send-to-queue in the flow (IV-01).
- **VO-08** Business rules on the voice stack only: one thing at a time, at most two short sentences; answer a question that comes with a yes; for an identifier, list what the caller has and let them choose by number; ask before you transfer (this is the only way to get ask-before-transfer on a call today); before ending, ask once if there is anything else; never say a filler such as "one moment".
- **VO-09** Codes on a call follow `voiceCallerVerificationEnabled`. When it is false: never mention a code. When it is true: say a text is on its way only after the tool answered, then wait for the caller.
- **VO-10** Bind the stack to the number: IV-06 in the `atender-ivr` skill.

Field names, language-code forms and the transfer decision are in
`references/voice-fields.md`.

## What does not apply on a call today

On a voice stack only `handoverTeamId`, `voiceHandoverAiTeam`,
`voiceFallbackQueueId` and `voiceFallbackAction` are read. These are inert on a
call: `handoverInstructions`, `handoverWhenUnsure`, `handoverAskConfirmation`
(ask before transfer), `handoverOfferEmailFollowup`,
`handoverOfferCloseAndReturn`, `handoverCheckOpeningHours`, the handover
required-information rows, and stack prerequisites.

Check the live schema on `create_agent_stacks` and `update_agent_stacks` before
you tell a customer any of them is inert — if the schema now reads them on voice,
follow the schema and say so. Do not store a setting that has no effect on the
channel.

*This changes when the app ships these as tenant settings on voice; re-check the
live schema.*

`handoverAskConfirmation` is not read on a call today. To make the assistant ask
before it transfers, write the confirmation rule into the voice stack's
personality `businessRules` (VO-08). That is the only place it takes effect.

*This changes when the app ships ask-before-transfer as a tenant setting on
voice; re-check the live schema.*

## Rules

- The stack voice map has no fallback. An empty map — from a PATCH `null`, a type flip or a clone — makes every call hear "this number is unavailable".
- Never promise a mechanism you have not proved on a call. The words the assistant says are not evidence that anything ran.
- A voice setting change and a stack or number change are read by the first call more than 30 seconds later.
- Never `create_conversations_inbound` to test a voice stack. It is an email path.
- A test call is a real call that costs money. Get a yes before each one, and do not score the first call.

## Verify

- Read `voiceLanguageVoices` back with `get_agent_stacks` after the create, and say which voice each language got.
- On each call check: the greeting (language, voice, not clipped); at most two sentences per turn; no talking over the caller; the question that comes with a yes is answered; nothing about the account before verification; a transfer reaches the queue named; the fallback path.
- The transcript is what the caller heard. Compare Norwegian calls with Norwegian calls. Speech recognition treats the language as a bias, not a lock, so one wrong-language answer is not a setting fault.
- A call writes no verification event. Report a call as not proved by event; read the event feed for the call as the record instead.
- Read the stack back with `get_agent_stacks` and compare `voiceLanguageVoices`, `voiceWelcomeGreetings`, `voicePace`, `voiceFallbackAction` and the transfer target field by field.

## What must be done in the app

Turning the AI voice feature on (Atender only, VO-01). Requesting a phone number,
which costs money. Recording decisions about call recording, which is the
customer's legal call. Tell the customer which of these are waiting on them.
