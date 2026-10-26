---
name: atender-web-chat
description: Set up or audit the Atender chat widget and other live web channels over the Atender MCP server — the widget's Agent Stack and team, languages, welcome message, after-hours message, the install snippet, custom channels that reach the customer's own application over a webhook, and branding. Use this skill whenever the user asks to put Atender chat on their website; when they say "chat widget", "the bubble on our site", "install the chat script", "chat goes to the wrong team", "after hours message", "connect our own app to Atender", "custom channel", or "webhook channel"; and whenever an Atender MCP connection is present and the work is about a visitor talking to the assistant on a web page or inside the customer's own product. Email inboxes and domains are the atender-email skill.
---

# Atender web chat

## When to use

Use this when a visitor should be able to talk to an Atender assistant from a web
page, or when the customer's own application should carry the conversation over a
webhook. It owns the chat widget, custom channels, the install snippet, the test
page, after-hours behaviour and the branding a visitor sees.

Email inboxes, email domains, DNS and SMS sender names belong to
`atender-email`. What the assistant actually says belongs to
`atender-agent-stack-text`.

## Before you start

Read `../_shared/atender-setup-basics.md` first — connection, precedence, the
order of areas, the preflight, pacing, the report format and the groundwork
areas. Then call `describe_configuration_model` with `area: "web-chat-email"`;
this skill wins on order and on how to test.

The order is: teams, then the Agent Stack, then the widget or channel, then a
test page, then the real page. A channel with no stack gets no AI answer, and
nothing tells you.

## Needs from the customer

- Which team answers web chat, and which Agent Stack.
- The languages the widget offers, and the welcome message in each.
- Opening hours for web chat, and what a visitor gets outside them.
- For their own application: a public HTTPS address for replies, or none.
- Where the widget goes: a test page first, then the real page.

## Checklist

- **CH-01** For each channel, report its Agent Stack (`mainAgentId`; on the widget `aliMainAgentId` and `afterHoursMainAgentId`), team and status. A channel with no stack gets no AI answer.
- **CH-05** `create_chat_widget`: `defaultTeamId`, `primaryLanguage`, `secondaryLanguages`, `welcomeMessage`, and the stack: `aliMainAgentId` (answers at all times) or `afterHoursMainAgentId` (answers only while the `defaultTeamId` team is closed). `aliEnabled` follows those two fields. The widget is live the moment it exists. After hours: an opening-hours rule on the primary team for web chat, and `outsideHoursMessage`.
- **CH-11** Texts per language: the widget fields hold the primary language. For each secondary language, `generate_chat_widget_translations` {`languages`} starts an AI translation (with `force` it redoes all and replaces hand edits); `list_chat_widget_translations` reads them; `update_chat_widget_translation` changes texts of one language (`null` removes one text); `delete_chat_widget_translation` removes the stored texts of one language, and the widget still offers it.
- **CH-06** No tool returns the install tag. Build it by hand from the widget id: `<script src="https://v3.atender.com/chat-widget.js" data-widget-id="<id>" async></script>`. It answers as soon as it loads, so put it on a page nobody visits first.
- **CH-07** Custom channel: `create_channels` with `mainAgentId` and `defaultTeamId`. Give the customer the `secret` once. Then `test_channels`, which POSTs one signed test envelope to `webhookUrl`; on a channel with no webhook it answers `ok: false` by design.
- **CH-09** Brand: `update_branding`, the widget colours, and `update_email_brand_settings` where the widget shares brand settings.
- **CH-10** Help centre, status page or portal on the customer's domain: `create_custom_domains` {`host`, `product`}. Give them the A record, and verify with `create_custom_domains_verify` after they publish it.

Widget fields and the custom-channel contract:
`references/widget-and-custom-channels.md`.

## Rules

- A channel answers only when its stack is enabled. Teams come first, then the stack, then the channel.
- **To take the AI off a widget: `update_chat_widget` with `aliMainAgentId: null` and `afterHoursMainAgentId: null`** — both, or the after-hours stack keeps answering. The widget stays up and people still get the chats.
- There is no public delete route for a widget, on purpose. A widget is deleted in the app. Create one deliberately.
- `set_opening_hours` replaces the whole week. Send `timezone` as an IANA name; the default is UTC.

## Verify

- `get_chat_widget`: `aliMainAgentId`, `afterHoursMainAgentId`, `defaultTeamId`, `primaryLanguage`, `secondaryLanguages`, `welcomeMessage` and `outsideHoursMessage` as planned. `list_chat_widget_translations`: a text for each secondary language.
- Open the widget on the test page. Record the widget session id when you open it, because an anonymous conversation is not on the conversation list. Time the gap from the stored reply to the screen.
- Custom channel with a webhook: `test_channels` passes, and `list_channel_deliveries` shows `success`.
- Outside hours: one visit shows the after-hours message; inside hours the assistant answers.
- `list_channels` and `list_chat_widgets`: every channel names a stack and a team.

## What must be done in the app

Putting the script tag on the real page. Deleting a widget. Publishing the A
record for a custom domain. Editing the portal layout. Tell the customer which
are waiting on them.
