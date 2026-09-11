---
name: atender-audit
description: Read a whole Atender workspace over the Atender MCP server and report what is set up and what is missing, against every checklist in this plugin, without changing anything. Use this skill whenever the user asks for a review rather than a change; when they say "inspect my Atender workspace and report", "change nothing", "audit our setup", "what is missing", "why is the assistant not answering", "health check", "review before we go live", or "give me a report I can hand to someone"; and whenever an Atender MCP connection is present and the work is to look, not to write. To fix what this finds, use the skill that owns the area.
---

# Atender audit

## When to use

Use this when the customer wants to know what their workspace looks like and
what is missing, and nothing is to change. It reads every area, compares what it
finds against the checklists the other skills apply, and writes one report the
customer can hand to whoever does the fixing.

It is the only-inspect mode of this plugin. Every fix belongs to the skill that
owns the area: `atender-workspace`, `atender-agent-stack-text`,
`atender-agent-stack-voice`, `atender-capabilities`, `atender-web-chat`,
`atender-email`, `atender-ivr`.

## Before you start

Read `../_shared/atender-setup-basics.md` first — connection, precedence, the
preflight, pacing and the report format. Then call
`describe_configuration_model` and note where it disagrees with what you read.

This skill writes nothing. Say that to the customer in the first message, and
again at the top of the report.

## Needs from the customer

Nothing. Ask only what the workspace cannot tell you, and only after the report
exists: which stack is meant to be live, which team is meant to answer what,
and whether a gap you found is on purpose.

## Checklist

- **AU-01** Say up front that this run writes nothing, and name the date of the read.
- **AU-02** Read every area with the list and get tools in `references/read-map.md`, one call per line, at most one call every 2 seconds.
- **AU-03** Record every read that answered 403, with the scope in the message, and list them at the top of the report. A refused read is unreadable, never a gap.
- **AU-04** Report per area, in the basics file's inspect format: `met | gap (what is missing) | unreadable (why)`, keyed to the checklist ids of the skill that owns the area — `WS-`, `KB-`, `HB-`, `AS-`, `SP-`, `PE-`, `HO-`, `CV-`, `TC-`, `CA-`, `CH-`, `VO-`, `IV-`.
- **AU-05** Walk the ten common faults below explicitly. Each gets a line whether or not it is present.
- **AU-06** For each gap, name the skill that fixes it and the one tool that does it. Do not write a plan of calls; that is the owning skill's job.
- **AU-07** Rank the gaps by what they stop: nothing reaches a customer, then a customer gets a worse answer, then a setting is untidy.
- **AU-08** Output one markdown report. Nothing else.

## The ten faults to look for

1. Knowledge Base articles that sit at `draft`. Only `published` reaches the AI.
2. A specialist that must follow policy with `handbookEnabled` false.
3. An Agent Stack that is `enabled` with no enabled member. It answers nothing.
4. A stack with `handoverMode: "explicit_team"` and no `handoverTeamId`.
5. An email channel or custom channel with no `mainAgentId`, or a widget with no `aliMainAgentId`. It gets no AI answer, and nothing says so.
6. A voice stack with no `voiceLanguageVoices` for a language it is meant to answer in.
7. No call queue, while a phone number or IVR flow exists. The number never syncs.
8. An IVR flow with a node that has no timeout edge. A caller who says nothing falls off the flow.
9. Tags created and never used, and tags the areas apply that do not exist. An unknown tag name is dropped with a 200.
10. A widget, channel, number or queue that names a stack that is disabled or gone, and a stack that nothing names at all.

## Rules

- **No write tool of any kind.** Not a create, not an update, not a delete, not a publish, not an enable, not a test. A test conversation is a write and a cost. If the customer asks for one, hand the run to the skill that owns the area.
- Do not filter a list you use to decide what exists. `list_kb_articles` takes no `status` filter, `includeArchived=true`, and every page.
- Match on the natural key — a name, a `slug`, an `externalId` — never on an id you did not create.
- A read that works does not prove write access. Never report write access as met.
- A 404 can be a module that is off rather than a fault. Say which of the two you are looking at, or say you cannot tell.
- Never guess a value you could not read. An unreadable area is reported as unreadable, with the status code.
- Put today's date on every "this does not exist" finding.
- After 3 failed attempts on one read, stop and record it.

## Verify

- Every line in `references/read-map.md` has a result: a value, a 403 or a 404.
- Every checklist id in the plugin appears in the report exactly once.
- Every one of the ten faults has a line.
- Every gap names the skill and the tool that fixes it.
- No write tool appears in the transcript of the run.

## What must be done in the app

Nothing here — this skill changes nothing anywhere. Say in the report which of
the gaps can only be closed in the app rather than over the API: inviting a
user, deleting a team, switching on AI auto-tagging per tag, fonts, uploading an
image file, the Knowledge Base portal layout, removing a custom domain,
publishing DNS records, requesting an SMS number, and entering any real
credential.
