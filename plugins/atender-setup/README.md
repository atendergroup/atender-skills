# Atender Setup

A Claude Code plugin that sets up and audits an Atender workspace over the
Atender MCP server. Eight skills, one per area of the product, built from the
setup guide at https://www.atender.com/docs/mcp-setup (evidence date
2026-09-10).

## What's inside

| Skill | What it owns |
| --- | --- |
| `atender-workspace` | The ground the workspace stands on: the company profile the other skills reuse, the brand on every surface, teams, tags, opening hours, the satisfaction survey, the SMS sender name and the signature |
| `atender-agent-stack-text` | The Agent Stack for email, chat, SMS, WhatsApp, Messenger and custom channels: the stack and its members, specialists, personality, handover to people, customer verification, test conversations |
| `atender-agent-stack-voice` | The voice Agent Stack: stack type, a voice per language, greeting, pace, transfer queues, fallback, and what handover and verification do differently on a call |
| `atender-capabilities` | Connecting the customer's own API: parse, endpoints, identity, tiers, redaction, create, publish, assign |
| `atender-web-chat` | The chat widget, custom channels, install snippet, test page, after-hours, branding |
| `atender-email` | Sending domains, DNS, the MX warning, inboxes, deliverability, SMS sender name, signatures |
| `atender-ivr` | Call queues, phone numbers, the flow graph, retries and timeouts, opening hours, publish, bind, test |
| `atender-audit` | Reading the whole workspace and reporting the gaps against every checklist here, changing nothing |

Every skill starts by reading `skills/_shared/atender-setup-basics.md`, which
carries the shared ground below.

## The Atender MCP server

Atender is a customer operations platform. The MCP server exposes the workspace
configuration as tools: teams, tags, Knowledge Base, Handbook, Agent Stacks,
specialists, Capabilities, channels, phone numbers, queues and IVR flows. Every
call acts on the one workspace the connection belongs to.

- MCP server: `https://v3.atender.com/api/v1/mcp`
- Docs: `https://www.atender.com/docs`
- API reference: `https://www.atender.com/docs/api` and `https://v3.atender.com/api/openapi.json`
- Scopes: `https://v3.atender.com/.well-known/oauth-protected-resource`

Connect by signing in:

```bash
claude mcp add --transport http atender https://v3.atender.com/api/v1/mcp
```

or with a workspace API key that starts with `sa_live_`, sent as an
`Authorization: Bearer` header. Never paste a key into a chat.

## What the skills agree on

- **Precedence.** The live tool input schemas win for field names and shapes. These skills win over `describe_configuration_model` for the order of areas and for how to test. Call `describe_configuration_model` first anyway, and report where it disagrees.
- **Order.** Teams, tags, Knowledge Base and Handbook, specialists, Agent Stack and members, personality, Capabilities, verification settings, opening hours, handover, channels, the voice Agent Stack, call queues and IVR flow and phone number, test conversations.
- **Preflight.** List what exists in every area before writing anything: teams, tags, Knowledge Base, Handbook, stacks, specialists, channels, numbers, queues.
- **The loop.** Inspect, ask, plan, wait for approval, apply, verify. Never delete or overwrite an existing object unless the user says yes to that object by name.
- **Pacing.** One call every 2 seconds. A test call is a real call that costs money.
- **The report.** Every checklist id, grouped by area: `done | skipped (reason) | blocked (reason)`, or `met | gap | unreadable` for an inspect-only run. Then the ids created, the steps that must be done in the app, and the open items with today's date.

The Knowledge Base and Handbook checklists live in
`skills/_shared/atender-setup-basics.md`, because no single skill owns them.
Teams, tags, opening hours and the brand belong to `atender-workspace`, which is
the first skill to run on a new workspace.

## Notes for readers

Sections marked `<!-- site:skip -->` are left out of the published guide on
atender.com; assistants using the skills should read them.

## Install

The marketplace manifest is at the repo root, so the whole repo is the
marketplace:

```bash
claude plugin marketplace add atendergroup/atender-skills
claude plugin install atender-setup@atender-skills
```

## Use

Once installed, ask Claude things like:

- "Set up my Atender workspace"
- "Configure the Agent Stack that answers our support inbox"
- "Connect our order API so the assistant can look up an order"
- "Put the Atender chat widget on our site"
- "Build the phone menu: press 1 for sales, 2 for support"
- "Inspect my Atender workspace and report — change nothing"

The right skill loads itself.
