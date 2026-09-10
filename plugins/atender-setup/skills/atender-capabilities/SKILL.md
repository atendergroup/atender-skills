---
name: atender-capabilities
description: Give an Atender AI assistant the ability to read and change data in the customer's own systems, over the Atender MCP server — parse an API description into a connection, add endpoints, bind the customer's identity, set the tier, redact the response, create and publish each Capability, and attach it to the specialists that need it. Use this skill whenever the user asks to connect Atender to their CRM, order system, billing, booking or internal API; when they say "give the AI a tool", "let it look up an order", "let it cancel a booking", "connect our API", "why can't the assistant see the customer's data", "publish a capability", or "the AI says it did something but nothing happened"; and whenever an Atender MCP connection is present and the work is about the assistant acting on another system.
---

# Atender Capabilities

## When to use

Use this when an Atender AI assistant has to look something up in, or change
something in, the customer's own system. A Capability is one operation of that
system, bound to a verified customer identity, at a tier that says what it may
touch, published so it becomes a tool, and held only by the specialists that need
it.

Use it also to diagnose: a reply that says "done" with nothing changed, a refusal
that names a tool the specialist already holds, or an assistant that cannot see a
customer's data.

## Before you start

Read `../_shared/atender-setup-basics.md` first — connection, precedence, the
order of areas, the preflight, pacing, the report format and the groundwork
areas. Then call `describe_configuration_model`. The live tool schemas win for
field names and shapes; this skill wins over `describe_configuration_model` for
the order of work and for how to test. `describe_configuration_model` is wrong
for Capabilities today: it points at `create_agent_tools`, which is only for raw
tools. The order below is the right one.

Verification settings come before any Capability above the `read` tier. The
specialists that will hold the tools come before the attach.

## Needs from the customer

- Where each API description is.
- For each operation: what happens if it goes wrong, in one line.
- The argument that identifies the customer, and whether it accepts a verified email or E.164 phone. If not: an identifier the customer holds, and where the API lists the people on that account.
- A throwaway credential. The real one is entered in the app.

## Checklist

- **CA-01** `create_api_definitions_parse`, then `create_api_definitions` with `name`, `baseUrl`, `authMode`, `authType`, `endpoints`. If the spec is too large, trim response schemas only.
- **CA-02** Each request body is `{contentType, schema}`, an OpenAPI `requestBody`, or a JSON Schema. Any other shape gives the model no `body` argument.
- **CA-03** Add missing operations with `create_api_definitions_endpoints`.
- **CA-04** Before any tier above `read`, set `authModeConfig` with `update_api_definitions`: `sessionVariableMapping`, `claimVariableMapping` (after a code the claim is the verified email or E.164 phone), `resourceOwnership` {`resourceKeys` = argument names, `lookup.path` with a `{{variable}}`, `lookup.identifierPaths`}. If the API cannot take that handle, check the live schema for `accountVerification`; if present, set it (unconfirmed on 2026-09-10; prove one bound call before you rely on it).
- **CA-05** Run `test_api_definitions_redaction` on a real sample response, then write `redactionRules`, `redactionApplyDefaults`, `redactionCustomFields`.
- **CA-06** `create_capabilities` for each operation: `name`, `displayName`, `description`, `usageGuidance` (when to call it, which inputs, what the answer means), `isSimpleTool: true`, `simpleToolConfig` {`actionType: "api_call"`, `apiDefinitionId`, `endpointId`}, `actionTier`.
- **CA-07** `actionTier`: see the tier table in `references/tiers-and-changes.md`. Do not send `securityLevel`. Set `ownershipExempt: true` only when the operation names nothing a customer owns.
- **CA-08** Show the customer a table: operation, tier, consequence, specialists.
- **CA-09** `publish_capabilities`. Fix each failed item, then publish again.
- **CA-10** `create_specialists_tools` with `toolId` = the Capability id, only where it is needed.
- **CA-11** To change a live Capability: see the change matrix in `references/tiers-and-changes.md`.
- **CA-12** Every "do X" rule has a specialist that holds the tool for X. A specialist that must choose one of several items holds the tool that lists them.

The identity model — session mapping, claim mapping, resource ownership — is in
`references/identity-model.md`.

## Rules

- Identity comes only from the verified session. Never from the contact or the conversation.
- A resource key names an argument, not a field in the answer. A connection holds one ownership rule.
- A mapped argument is filled only after verification, so the Brief says: verify first.
- A refresh of the documentation deletes the non-manual endpoints that left the spec.
- The publish checklist runs only when a Capability becomes published. After a connection change, unpublish and publish one Capability to run it again.
- Unpublish removes the tool from every specialist and keeps the assignments.
- If a refusal names a Capability that is attached and published, check the assignment once, then report a platform fault. The gap is the runtime tool surface, not the wording.
- A create with status `published` is refused. Create, then publish.
- Read a refusal properly: the code and `details` name the failing item.

## Verify

- `get_capabilities`: `published`, with `agentToolId` and `publishedAt` set.
- `get_agent_tools` on `agentToolId`: `description` and `inputSchema` are what the model reads.
- `list_specialists_tools` matches the table from CA-08.
- After a real conversation, `list_tool_execution_logs` with `conversationId` shows `success` and the `inputPayload`, then `get_tool_execution_logs` for the detail. A reply that says "done" is not proof.
- Every action a Brief promises has a published, attached tool.

## What must be done in the app

Entering the real credential — never in the chat. Connecting another MCP server
or a code repository, which has no API. Tell the customer which of these are
waiting on them, and where.
