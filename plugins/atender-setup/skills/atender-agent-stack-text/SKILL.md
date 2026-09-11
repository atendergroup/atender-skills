---
name: atender-agent-stack-text
description: Set up or audit an Atender Agent Stack for text channels over the Atender MCP server — the stack itself and its members, the specialists behind it, its personality, handover to people, customer verification, and the test conversations that prove it. Use this skill whenever the user asks to build, configure, fix, review or switch on an Atender AI assistant for email, chat, SMS, WhatsApp, Messenger or a custom channel; when they say "set up my Atender workspace", "configure my Agent Stack", "add a specialist", "why does it hand over", "change the tone", "it says the wrong thing", or "test my assistant"; and whenever an Atender MCP connection is present and the work is about what the AI answers on a text channel. Not for voice — use atender-agent-stack-voice for calls.
---

# Atender Agent Stacks: text channels

## When to use

Use this when the customer wants an Atender AI assistant that answers on text
channels — email, web chat, SMS, WhatsApp, Messenger or their own application —
and when they want to change, review or prove one that already exists. It owns
the stack, its specialists, its tone, when it steps aside to a person, when it
makes a customer prove who they are, and how you test it without reaching a real
customer. Calls are a different area: use `atender-agent-stack-voice`.

## Before you start

Read `../_shared/atender-setup-basics.md` first — connection, precedence, the
order of areas, the preflight, pacing, the report format and the Knowledge Base
and Handbook groundwork. Then call
`describe_configuration_model`. The live tool schemas win for field names and
shapes; this skill wins over `describe_configuration_model` for the order of work
and for how to test.

Teams, tags, the Knowledge Base and the Handbook come before anything here.
`atender-workspace` owns the first two.

## Needs from the customer (or from `atender-profile.md`)

- What customers write in about, in the customer's own words.
- Which job answers when nothing else fits.
- For each job: what it may do on its own, with the limit as a value.
- At least one team that picks up a handover, and one channel this stack answers on.

Tone, handover criteria, opening hours, verification and test situations are
asked for in their own sections.

## Checklist: the stack

- **AS-01** Inventory. Per stack: `name`, `type`, `enabled`, `catchAllSpecialistId`, members and whether each is enabled, `kbPartitionId`, `handbookPartitionId`, `knowledgeBaseEnabled` (voice only, AS-05), orchestrator `resolvedSource`. Name the channels, email channels, phone numbers and SMS numbers that carry it in `mainAgentId`. Report a stack nothing points at.
- **AS-02** Stack count = one per job, not one per channel. Voice gets its own stack.
- **AS-03** `create_agent_stacks`: `name` (unique, the natural key), `type`, `enabled: false` — the default is true, so a stack is live from its create call. Order: **create disabled, add specialists and members, bind the channel, then `enable_agent_stacks`.** A stack enabled earlier answers with no members behind it.
- **AS-04** Bind partitions: `kbPartitionId` and `handbookPartitionId` from `list_kb_partitions` / `list_handbook_partitions`, even with one of each. A later `kbPartitionId` change drops the members' category limits.
- **AS-05** `knowledgeBaseEnabled` is a **voice** switch and gates nothing on text. Text retrieval is per specialist: `kbEnabled` and `handbookEnabled` (SP-04). Do not set it on a text stack, and do not report it as a fault there.
- **AS-06** Before any member is added: each specialist exists, is enabled, and has its final description (SP-03).
- **AS-07** `create_agent_stacks_members` {`agentId`, `role: "agent"`}, one call each, the intended catch-all first. The first enabled specialist added becomes the catch-all.
- **AS-08** The orchestrator routes on each member's description. It must be final before the add.
- **AS-09** `catchAllSpecialistId` = the intended specialist. To correct it: `update_agent_stacks` {`catchAllSpecialistId`}. It must be an enabled member.
- **AS-10** Orchestrator: read first. `update_agent_stack_orchestrator` {`systemPrompt`} only for stack-wide routing the descriptions cannot carry; it adds to the built-in prompt.
- **AS-11** Handover, verification, personality and specialists: `references/specialists-and-personality.md` and `references/handover-and-verification.md`.
- **AS-12** Enable gate, last. The route checks nothing, so you check: AS-03 to AS-11 done, the specialists of SP-03 and SP-04 in place, HO-01 and HO-02 set, one channel bound to this stack, and one test conversation passed (TC-01). Then `enable_agent_stacks`. Way out: `disable_agent_stacks`.

### The minimum a lone stack needs

When the run covers the stack and nothing else, it still has to be complete:

- **AS-M1** One specialist, before AS-06. If none exists: `create_specialists` {`displayName`, `systemPrompt`, `description` = one sentence on what it owns, as a criterion}. `handbookEnabled: true` where it must follow policy; `enabled` and `kbEnabled` default to true. The description is final before the member add, and the first enabled member becomes the catch-all.
- **AS-M2** A handover target, in one `update_agent_stacks` call before the enable: `handoverMode: "explicit_team"`, `handoverTeamId` = a team from `list_teams` (`create_teams` if there is none), `handoverAskConfirmation: true`. If there is no handover yet, set `handoverMode: "never"` explicitly and report AS-M2 as a gap: the stack then never gives a conversation to a person.
- **AS-M3** One channel bound to this stack, with the customer's yes: `update_email_channels` {`mainAgentId`}, `update_chat_widget` {`aliMainAgentId`}, or `create_channels` / `update_channels` {`mainAgentId`}. Say which stack each channel leaves. No channel yet: report AS-M3 as a gap, nothing reaches the stack.

### The safe enable test, and how to read a trace

The safe path is a **pull channel**: a custom channel with no webhook, so there
is nowhere a reply can be pushed and nothing reaches a customer. After AS-12:

1. `create_channels` {`type: "custom"`, `mainAgentId` = this stack, `defaultTeamId`, **no `webhookUrl`**}.
2. `create_channels_messages` with the customer's first message.
3. Read the trace: `list_conversations_messages`, `list_conversation_events`, `list_routing_decisions` (specialist, confidence) and `list_tool_execution_logs` (what ran and what it returned).

The conversation is real and counts in analytics: keep the batch small and tag
it. Lighter alternative: `test_specialists` {`mainAgentId` = this stack} on
three real situations — wording and routing only, no conversation, no handover
commit, no routing decision. Never `create_conversations_inbound` here: every AI
reply is a real email.

<!-- site:skip -->
## The other checklists

| Reference | Ids |
| --- | --- |
| `references/specialists-and-personality.md` | SP-01 to SP-10, PE-01 to PE-10 |
| `references/handover-and-verification.md` | HO-01 to HO-08, CV-01 to CV-08 |
| `references/test-conversations.md` | TC-01 to TC-13 |

## Rules

- A stack write refuses an unknown key with a 400.
- `null` on `systemPrompt` deletes the stack's rules. If `stackOverride` is null and nothing is needed, send nothing.
- The default stack cannot be deleted, and `isDefault` cannot be written.
- Never add a member with `role: "router"`.

## Verify

- `get_agent_stack_orchestrator`: `resolvedSource` and `resolvedRules`.
- `get_specialists` on each member: the description reads as intended.

## What must be done in the app

AI auto-tagging per tag, enabling languages, restricting a specialist to some
Knowledge Base categories, adding a user who is in no team, entering a real
credential. Tell the customer which are waiting on them.
