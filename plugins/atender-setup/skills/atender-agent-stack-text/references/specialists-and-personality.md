# Specialists and personality

## Specialists (SP)

Goal: one specialist per separate job, each with a routing description, the right
knowledge switches, the tools its Brief promises, and a Brief the model follows.

- **SP-01** Inventory. Per specialist: `displayName`, `enabled`, `kbEnabled`, `handbookEnabled`, the stacks it is on, tools, playbooks. A specialist on two stacks is one shared record. List the stacks and wait before any change.
- **SP-02** Propose the set yourself: one per separate job, six to ten is typical. Name the catch-all.
- **SP-03** `create_specialists`: `displayName` = the natural key. `description` = one sentence on what it owns, as a criterion. It becomes its routing topic (AS-08).
- **SP-04** `kbEnabled: true` (default true). `handbookEnabled: true` on every specialist that must follow internal policy. The default is false.
- **SP-05** Leave `defaultTargetAgentId` and `isKnowledgeAgent` unset. The text path does not read them.
- **SP-06** `systemPrompt` says: the order of work, numbered; its limits, each as an action it cannot take, with values, never as a state the item is in; the tools it uses at each step. The reply shape and house style go in personality. The handover criteria go in handover. For a specialist that holds a level 2 Capability, add the CV-07 wording.
- **SP-07** `create_specialists_tools` {`toolId` = a published Capability id} for every action the Brief says it does. Tool coverage and publishing: the `atender-capabilities` skill.
- **SP-08** Playbooks, if a job has stages: `create_specialists_playbooks` {`name`, `canvas`, `maxIterations`}. It belongs to one specialist, and a copy drifts. After a Capability change, send a PATCH to each Playbook that quotes it, then read `synthesizedProse` back.
- **SP-09** Specialist routing rules, if a conversation moves between jobs: `create_specialists_routing_rules` {`label`, `llmTopic` as a criterion, `targetAgentId`, `priority`}. The target must be on the same stack.
- **SP-10** Add to the stack: AS-06 and AS-07.

Rules:

- Never put a real date, name, amount or address in an example. Use placeholders, and say that every date, time and address comes from a tool.
- If you want paragraphs, say the word paragraph. Never write "no more than N sentences".
- Never tell a specialist to volunteer what the service does not include.
- When one step depends on another, say who owns the next step in the same paragraph. A dependency sentence reads as an order to hand over.
- Never edit a shared specialist to fix one stack. Tone lives on the stack personality.

Verify:

- `list_specialists_tools`: every tool the Brief names is held and enabled.
- `list_specialists_playbooks`: `synthesizedProse` is filled, and it says what you meant.

## Personality (PE)

Goal: each stack's tone is explicit configuration, chosen from generated replies.

- **PE-01** Read `personality` on every stack. Show every field, and each context field's length against its cap.
- **PE-02** In the same write as `persona`, send all six dimensions. `humor` = `off` unless the customer asks.
- **PE-03** `firstPerson` = true, unless the customer says no.
- **PE-04** `useCustomerName` = decided on purpose, and say the value.
- **PE-05** `replyLength` = the customer's answer. Set `lengthByChannel.short` (chat, messaging) and `.long` (email) only where they differ.
- **PE-06** `language` = `"auto"` or one BCP-47 tag. In `businessRules`, add: "Judge the language from the words the customer wrote, never from a name, city, address, phone number or code."
- **PE-07** `personality.context`: `businessDescription` = what the company does. `customerDescription` = who writes in, in which languages. `businessRules` = house style and rules that apply to this stack only. Do not copy Handbook entries. `custom` = the reply shape, in paragraphs, with a blank line between them.
- **PE-08** Preview 3 candidates, each on `refund_policy`, `frustrated` and `off_topic`. Show the replies side by side and recommend one in two lines.
- **PE-09** Save the chosen candidate with `update_agent_stacks`.
- **PE-10** Read back with `get_agent_stacks`, then preview with `sampleMessageId` and no `personality`.

Rules:

- `context` goes inside `personality`, never beside it.
- `null` on a `context` or `lengthByChannel` key deletes it. Omit a key to keep it.
- A preview judges tone only. It uses a fixed sample, chat length, no Brief, no Knowledge Base, no tools, and a one-hour cache.
- Tone and reply shape live here once. A Brief states only what differs, and a shared specialist never gets one stack's tone.
- No greeting or signature field exists here. `set_default_signature` is a person's signature, not the AI's.

Verify:

- The preview from PE-10 matches the chosen candidate.
- List each rule that is in `businessRules` and also in a Brief, and name the owner.
