# Handover to people, and customer verification

## Handover (HO)

A handover is the AI stepping aside; a takeover is a person taking the
conversation, and nothing here configures one.

Goal: a conversation reaches a person only on the criteria the customer named, in
a team that exists, with the facts that team needs.

These items are for text stacks. On a voice stack only HO-02 applies; the
`atender-agent-stack-voice` skill owns the rest.

- **HO-01** `handoverMode` = `explicit_team` (one team) or `by_description` (the AI picks among teams that have a description). Set `never` only if the customer says so in writing.
- **HO-02** `handoverTeamId` = an existing team id. Required with `explicit_team`, but the API does not refuse it when missing: the handover then fails at run time, and the error is logged in the message debug panel in the app — look there for a conversation that should have handed over and did not. With `by_description`, every candidate team has a description.
- **HO-03** `handoverInstructions` (per stack, never copied): a paragraph saying handing over is the last step (identify the customer, read the record, do everything inside your limits first); then one criterion per line; then the ruled-out cases in words, for example "frustration is not a handover; the first two replies after a complaint begins are answered in full". Show the customer the full text before you save it.
- **HO-04** `handoverWhenUnsure` = false, unless the customer wants every message outside the specialists' topics sent to a person with no clarifying question. Same value on every text stack.
- **HO-05** `handoverAskConfirmation` = true. The customer then chooses "Yes, transfer me" or "Not yet" before the handover commits.
- **HO-06** Opening hours, in order: ask the customer for the timezone as an IANA name (`Europe/Oslo`), then `create_opening_hours_rule` with that answer in `timezone` and with `holidayCountry`, then `set_opening_hours_assignment` per team and channel. Never leave `timezone` out: the API stores `UTC` in silence, and the handover deferral reads the rule's own timezone, so the hours defer by the offset. Read each rule back and check `timezone` equals the answer. A pair with no assignment takes the default rule; no rule at all reads as always open.
- **HO-07** `handoverOfferEmailFollowup` and `handoverOfferCloseAndReturn` = the customer's answer. `handoverCheckOpeningHours` = true only after HO-06 and only with at least one offer true. With both offers false a closed team still gets the handover.
- **HO-08** One required-information row per fact the team needs: `fieldScope` and `fieldKey` that exist (conversation: `subject` or a conversation custom field; contact: `email`, `phone` or a CRM field); `fieldLabel` = a noun phrase under 60 characters that completes "Before I connect you with a teammate, I need one more thing, {label}."; `askPrompt` = an instruction to the AI; `conditionMode` `always`, or `when` with `conditionText` written against what the customer said, never against a subject. Match existing rows on `fieldScope` and `fieldKey`, never on an id.

Rules:

- Every sentence in `handoverInstructions` is appended to the handover tool description and reads as an instruction to escalate. Write criteria, never subjects, and no customer copy.
- The reply text does not prove a handover happened or did not. Read the conversation record.
- **Always send `fieldScope: "conversation"` on `create_custom_field` when a requirement will name the field.** It defaults to contact, a contact-scope field cannot back a requirement, and the scope cannot be changed afterwards.
- Do not delete a custom field a requirement names. The requirement can then never be met.
- An email or phone requirement is met only by an address already trusted on the contact. A typed address makes the customer prove it with a code where a code can be sent.

Verify:

- Read every stack and its requirements back: one table, one row per stack, before and after.
- Per text stack: one conversation that must not hand over and one that must. Read the handover state, reason, team and each required field from the conversation record.
- If HO-07 is on: one handover outside hours shows the offers; one inside hours commits.
- The reply "I'm not able to connect you with a teammate right now" means no team was reachable: read the message debug panel for the logged error, then fix HO-01 and HO-02.

## Customer verification (CV)

Goal: the four verification switches match the customer's policy on each channel,
and every specialist that needs a verified customer asks for a one-time code,
never for an identifier.

Needs from the customer: on which channels a customer must be verified before the
assistant says anything about their account; whether a chat visitor may verify an
email address that is not on their record; whether the assistant may say a masked
destination such as "...5206"; whether it may send a code during a call; whether
their own application signs the customer in first.

- **CV-01** `revealPosture` = the customer's answer.
- **CV-02** `selfServiceEmailOtpEnabled` = the customer's answer (default false).
- **CV-03** `stepUpByCodeEnabled` = true wherever a level 2 Capability serves email, SMS, WhatsApp, Messenger or a custom channel.
- **CV-04** `voiceCallerVerificationEnabled` = true only if callers must reach level 2 Capabilities.
- **CV-05** Custom channels: `settings.verificationDelivery` = `contact`, unless the customer's application signs the customer in first; then redact the six digits in their own logs.
- **CV-06** Order: CV-01 to CV-05 before any Capability with `securityLevel` 2 goes on a stack. Level 0 needs no proof, 2 needs a code, 3 needs a signed-in customer. Give no level 2 Capability to a stack whose channel has no working method: voice with CV-04 false, or text with CV-03 false (the widget excepted).
- **CV-07** Every specialist that holds a level 2 Capability carries this wording, written into its `systemPrompt` by SP-06: "Verify with a one-time code before anything else. Never ask the customer for the email address or phone number on the account. An identifier the customer types proves nothing."
- **CV-08** Read a sample of contacts. A code goes only to an address with provenance `transport`, `agent_entered`, `tenant_import`, `tenant_api` or `verified`; never `customer_claimed` or `api_asserted`. Report how many contacts hold only typed addresses; they get no code, only a handover.

Rules:

- Changes apply from the next turn.
- Spoken code wording is in the `atender-agent-stack-voice` skill (VO-09).
- Do not state how long a code lasts. No setting reports it.

Verify:

- Per channel that verifies: prove a code, then ask a second question; it is answered, not challenged again. Once more where the customer volunteers the address first.
- On text channels, `list_conversation_events` shows `verification_requested` and `verification_completed`; no event means no code was requested. A call writes no verification event; report a call as not proved by event.
