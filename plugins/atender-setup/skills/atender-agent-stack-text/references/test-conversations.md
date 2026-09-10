# Test conversations (TC)

Goal: prove the stack on three or four real situations, on a path that reaches no
real customer, and read every result back.

Needs from the customer: the three or four most common situations, with the
customer's first message in their own words and how it should end; what must be
true in their own system for each one; a mailbox on a domain they control for
every test contact, for example `qa+01@yourdomain`.

- **TC-01** Pick the path for each situation, and say what it cannot reach.
  - a) `test_specialists` (`mainAgentId` = a stack the specialist is in; `conversationHistory` for later turns). It runs the router, specialist, output guardrails and real level-0 tools. No conversation, level 0, no handover commit, no routing decision. Wording checks only.
  - b) A custom channel with `mainAgentId`: `create_channels_messages`, with one `externalConversationId` per run. The reply goes to the channel's `webhookUrl` only. Check the receiver first with `test_channels`. This is the preferred proof.
  - c) Email: `create_conversations_inbound` with `emailChannelId` = an inbox with a stack, then `append_conversation_customer_message` for each later turn. Every AI reply is a real email to `contact.email`.
  - You cannot reach these over MCP: the chat widget, SMS, WhatsApp, Messenger, voice, test-marked conversations and the Agent Stack trace.
- **TC-02** Every `contact.email` or `sender.email` = a test mailbox the customer controls. Never a customer's address.
- **TC-03** Tags = one general test tag plus one per situation, created before the first run. Send them in `tags` on `create_conversations_inbound`. On a custom channel, set them with `update_conversations`.
- **TC-04** A new `externalReference`, `externalConversationId` and message `externalId` for each run. A repeated key returns the old result and runs no turn.
- **TC-05** Turn 1 = the start of the story, in the customer's words.
- **TC-06** Stage the account data the questions need, immediately before the run.
- **TC-07** 3 runs per situation.
- **TC-08** After each turn, poll `list_conversations_messages` with bounded retries. `agentStack: "queued"` is not a reply.
- **TC-09** Read back each run: `list_conversations_messages` (the reply in full), `get_conversations` (`handover`), `list_conversation_events`, `list_routing_decisions?conversationId=` (specialist, confidence, tools, guardrail results), `list_tool_execution_logs?conversationId=` then `get_tool_execution_logs`.
- **TC-10** Match each date, amount, address or action in a reply to a tool call or a tool log. List each claim that has no record.
- **TC-11** Review tone: quote, fault, and the setting or Brief that made it. Faults: one block, not paragraphs; passive with nobody in it; key point last; a feeling named back; an unasked exclusion; a date, time, address or amount no tool returned; the wrong language; a handover offer with no criterion met. Before you write a language rule, check that the sentence is the customer's and not platform text.
- **TC-12** One fix per re-run. Show before and after side by side.
- **TC-13** Clean up: `close_conversations` for each test. Conversations cannot be deleted. `delete_contacts` removes a test contact, but its conversations stay. Give the customer the `list_conversations?tag=` filter that finds them all.

Rules:

- Conversations made with `create_conversations_inbound` count in analytics, CSAT and billing. Keep the batch small.
- A harness never matches exact wording, and never answers a question the customer already answered.
- Demo: the presenter's own phone and email = one contact, renamed, not a second record. Present with a card of the identifiers.

Verify: one table — situation, run, path, specialist that answered, handover
state, claims with no record. Three rows for each situation.
