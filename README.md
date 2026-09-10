# Atender skills

Skills for AI assistants that set up an Atender workspace over the Atender MCP
server (`https://v3.atender.com/api/v1/mcp`). The repo is a Claude Code plugin
marketplace; it holds one plugin today, `atender-setup`, and has room for more.

Atender is a customer operations platform. The MCP server exposes the workspace
configuration as tools — teams, tags, Knowledge Base, Handbook, Agent Stacks,
specialists, Capabilities, channels, phone numbers, queues and IVR flows — and
every call acts on the one workspace the connection belongs to. The skills below
turn that surface into a guided setup: inspect what exists, ask, plan, wait for
approval, apply, verify, then report against a checklist.

## The skills in `atender-setup`

| Skill | What it owns |
| --- | --- |
| `atender-agent-stack-text` | The Agent Stack for email, chat, SMS, WhatsApp, Messenger and custom channels: members, specialists, personality, handover to people, customer verification, test conversations |
| `atender-agent-stack-voice` | The voice Agent Stack: stack type, a voice per language, greeting, pace, transfer queues, fallback, and what differs on a call |
| `atender-capabilities` | Connecting the customer's own API: parse, endpoints, identity, tiers, redaction, create, publish, assign |
| `atender-web-chat` | The chat widget, custom channels, install snippet, test page, after-hours, branding |
| `atender-email` | Sending domains, DNS, the MX warning, inboxes, deliverability, SMS sender name, signatures |
| `atender-ivr` | Call queues, phone numbers, the flow graph, retries and timeouts, opening hours, publish, bind, test |

Each skill starts by reading `skills/_shared/atender-setup-basics.md`, which
holds the connection, the order of areas, the preflight, the pacing and the
report format once, so the six skills agree.

## Install

```bash
claude plugin marketplace add atendergroup/atender-skills
claude plugin install atender-setup@atender-skills
```

Then ask for what you want — "set up my Atender workspace", "build the phone
menu", "connect our order API" — and the right skill loads itself. See
[`plugins/atender-setup/README.md`](plugins/atender-setup/README.md) for the
full picture.

## Where the content comes from

The skills mirror the setup guide at
<https://www.atender.com/docs/mcp-setup>. The field names, tool names and
behaviour in them were checked against the Atender API on 10 September 2026.

## Contributing

Lessons go to the setup guide first. Fix or extend
<https://www.atender.com/docs/mcp-setup>, then bring the change here so the
skills and the guide stay in step.
