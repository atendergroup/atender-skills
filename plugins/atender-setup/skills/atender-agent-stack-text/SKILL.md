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
order of areas, the preflight, pacing, the report format and the Teams, tags,
Knowledge Base and Handbook groundwork. Then call
`describe_configuration_model`. The live tool schemas win for field names and
shapes; this skill wins over `describe_configuration_model` for the order of work
and for how to test.

Teams, tags, the Knowledge Base and the Handbook come before anything here.

## Needs from the customer

- What customers write in about, in the customer's own words.
- Which job answers when nothing else fits.
- For each job: what it may do on its own, with the limit as a value.
- One paragraph they would send as it is, or two companies whose writing they like; house style no setting covers; reply length on chat and on email.
- What must always reach a person, as criteria, not subjects. Which team gets each kind. What that team needs in hand.
- Opening hours per team and channel, with timezone and holiday country, or "always open for now".
- On which channels a customer must be verified before the assistant says anything about their account.
- The three or four most common situations: the first message in the customer's words, and how it should end. A mailbox on a domain they control for test contacts.

## Checklist: the stack

- **AS-01** Inventory. Per stack: `name`, `type`, `enabled`, `catchAllSpecialistId`, members and whether each is enabled, `kbPartitionId`, `handbookPartitionId`, `knowledgeBaseEnabled`, orchestrator `resolvedSource`. Say which channels, email channels, phone numbers and SMS numbers name it in `mainAgentId`. Report a stack nothing points at.
- **AS-02** Stack count = one per job, not one per channel. Voice gets its own stack.
- **AS-03** `create_agent_stacks`: `name` (unique, the natural key), `type`, `enabled: false`. The default is true, so a stack is live from its create call.
- **AS-04** `kbPartitionId` and `handbookPartitionId` = explicit ids from the partition lists, even if there is one of each. A later `kbPartitionId` change drops the members' category limits.
- **AS-05** `knowledgeBaseEnabled: true`. Text reads the specialists' switches (SP-04).
- **AS-06** Before any member is added: each specialist exists, is enabled, and has its final description (SP-03).
- **AS-07** `create_agent_stacks_members` {`agentId`, `role: "agent"`}, one call each, the intended catch-all first. The first enabled specialist added becomes the catch-all.
- **AS-08** The orchestrator routes on each member's description. It must be final before the add.
- **AS-09** `catchAllSpecialistId` = the intended specialist. To correct it: `update_agent_stacks` {`catchAllSpecialistId`}. It must be an enabled member.
- **AS-10** Orchestrator: read first. Write `update_agent_stack_orchestrator` {`systemPrompt`} only for stack-wide routing criteria the descriptions cannot carry. It adds to the built-in prompt.
- **AS-11** Handover, verification, personality and specialists: `references/specialists-and-personality.md` and `references/handover-and-verification.md`.
- **AS-12** Enable gate. The route checks nothing, so you check: AS-03 to AS-11 done, the specialists of SP-03 and SP-04 in place, HO-01 and HO-02 set, one channel bound to this stack, and one test conversation passed (TC-01). Then `enable_agent_stacks`. Way out: `disable_agent_stacks`.

### The minimum a lone stack needs

When the run covers the stack and nothing else, it still has to be complete:

- **AS-M1** One specialist, before AS-06. If none exists: `create_specialists` {`displayName`, `systemPrompt`, `description` = one sentence on what it owns, as a criterion}. `handbookEnabled: true` where it must follow policy; `enabled` and `kbEnabled` default to true. The description must be final before the member add, and the first enabled member becomes the catch-all.
- **AS-M2** A handover target, in one `update_agent_stacks` call before the enable: `handoverMode: "explicit_team"`, `handoverTeamId` = a team from `list_teams` (`create_teams` if there is none), `handoverAskConfirmation: true`. If the customer says there is no handover yet, state `handoverMode: "never"` explicitly and report AS-M2 as a gap: the stack then never gives a conversation to a person.
- **AS-M3** One channel bound to this stack, with the customer's yes: `update_email_channels` {`mainAgentId`}, `update_chat_widget` {`aliMainAgentId`}, or `create_channels` / `update_channels` {`mainAgentId`}. Say which stack each channel leaves. If there is no channel yet, report AS-M3 as a gap: nothing reaches the stack.

### The safe enable test

With no test area in the run, one of these proves the stack without reaching a
customer: a custom channel whose `mainAgentId` is this stack, checked with
`test_channels` first, then `create_channels_messages`, so no reply reaches a
customer; or `test_specialists` {`mainAgentId` = this stack} on three real
situations, which reads wording and routing only, proves no handover, and runs
real level-0 tools. Never `create_conversations_inbound` here: every AI reply is
a real email.

## The other checklists

| Reference | Ids |
| --- | --- |
| `references/specialists-and-personality.md` | SP-01 to SP-10, PE-01 to PE-10 |
| `references/handover-and-verification.md` | HO-01 to HO-08, CV-01 to CV-08 |
| `references/test-conversations.md` | TC-01 to TC-13 |

## Rules

- A stack write refuses an unknown key. Build the body from the fields you change, never from a read.
- `null` on `systemPrompt` deletes the stack's rules. If `stackOverride` is null and nothing is needed, send nothing.
- The default stack cannot be deleted, and `isDefault` cannot be written.
- A member add writes an orchestrator routing rule from the specialist's description, once. Write the description before the add.
- Never add a member with `role: "router"`.

## Verify

- `get_agent_stack_orchestrator`: `resolvedSource` and `resolvedRules`.
- `get_specialists` on each member: the description reads as intended.
- Every planned field read back and compared, not a success response.
- One test conversation per stack, on a path that reaches no real customer.

## What must be done in the app

Switching AI auto-tagging on per tag, enabling languages, restricting a
specialist to some Knowledge Base categories, and adding a user who is in no
team. Entering a real credential. Tell the customer which of these are waiting
on them.
