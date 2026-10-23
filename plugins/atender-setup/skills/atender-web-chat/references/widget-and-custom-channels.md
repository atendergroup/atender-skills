# The chat widget, and custom channels

## Widget fields

| Field | What it does |
| --- | --- |
| `aliMainAgentId` | The Agent Stack that answers at all times |
| `defaultTeamId` | The team a conversation lands in, and the team whose opening hours the widget reads |
| `primaryLanguage`, `secondaryLanguages` | The languages the widget offers a visitor |
| `welcomeMessage` | The first thing a visitor reads, in the primary language. Other languages: the translation tools below |
| `outsideHoursMessage` | What a visitor reads when the primary team is closed |
| `afterHoursMainAgentId` | The Agent Stack that answers only while the `defaultTeamId` team is closed. With neither stack field the widget gets no AI answer |
| colours | Widget branding. The rest of the brand is `update_branding` |

## Texts in more than one language

The widget fields hold the primary language. The texts of each secondary language
are stored apart:

- `generate_chat_widget_translations` {`languages`, `force`} starts an AI translation of the stale languages; `force` redoes all of them and replaces hand edits.
- `list_chat_widget_translations` reads the texts per secondary language.
- `update_chat_widget_translation` changes some texts of one secondary language; `null` removes one text. The primary language is refused here.
- `delete_chat_widget_translation` removes the stored texts of one language. The widget still offers that language.

<!-- site:skip -->
## Taking the AI off a widget

There is no off switch on the widget itself. To stop the AI answering while the
widget stays up: `update_chat_widget` with `aliMainAgentId: null` **and**
`afterHoursMainAgentId: null`. Both, or the after-hours stack keeps answering.
Visitors can still start a chat and people still get those chats — only the AI
stops.

There is no public delete route for a widget, on purpose. A widget is deleted in
the app, not from here.

<!-- site:skip -->
## Routing

A widget conversation goes to `defaultTeamId` and is answered by
`aliMainAgentId`. `marketRouting` sends a market to a team. There is no per-page
routing on the widget: to send different pages to different teams, use more than
one widget, or route inside the Agent Stack with specialist routing rules.

<!-- site:skip -->
## After hours

1. `create_opening_hours_rule` with `timezone` as an IANA name such as `Europe/Oslo` — the default is UTC — and `holidayCountry`.
2. `set_opening_hours_assignment` for the primary team and the web chat channel.
3. `outsideHoursMessage` on the widget.

A team and channel pair with no assignment takes the default rule. No rule at all
reads as always open.

<!-- site:skip -->
## The install snippet

No route returns the install tag today. Build it by hand, with the widget id from
`get_chat_widget`:

```html
<script src="https://v3.atender.com/chat-widget.js" data-widget-id="<id>" async></script>
```

It answers as soon as it loads. Put it on a page nobody visits first — a staging
page, or a page behind a login — run the checks below, and only then hand it to
the customer for the real page.

<!-- site:skip -->
## The test page

- Open the page and start a conversation. Record the widget session id at the moment you open it: an anonymous conversation is not on the conversation list, so without the id you cannot read the run back.
- Check the welcome message, the language, and that a reply arrives.
- Time the gap between the stored reply and the reply appearing on screen.
- Close the tab and reopen it: check what the visitor sees on a second visit.
- Check the after-hours message by testing outside the team's hours, or by moving the rule for the test.

## Custom channels

A custom channel carries a conversation inside the customer's own application.

1. `create_channels` {`type: "custom"`, `mainAgentId`, `defaultTeamId`}. Add a `webhookUrl` only if replies must be pushed somewhere.
2. Give the customer the `secret` once — it is shown once. Never ask them to paste it back into the chat.
3. `test_channels` to prove the receiver before any message. A channel with no `webhookUrl` answers `ok: false` by design.
4. `create_channels_messages` to send an inbound message. The reply goes to the channel's `webhookUrl` only, so it reaches no customer.
5. `list_channel_deliveries` to read the result; `resend_channel_delivery` for a failed one.

### A pull channel, for testing

Create it with **no `webhookUrl`**: there is then nowhere a reply can be pushed.
The stack must be enabled first: a message on a channel runs a turn only on an
enabled stack. Prove the wording before the enable with `test_agent_stacks`.
Send with `create_channels_messages` and read the trace back with
`list_conversations_messages`, `list_conversation_events`,
`list_routing_decisions` and `list_tool_execution_logs`. This is the safe test
path (TC-01). The conversation is real and counts in analytics.

Facts to hold on to:

- A push channel turns itself off after 10 failed deliveries in a row. A resend counts as a new delivery. `update_channels` `isActive: true` turns it on again.
- `rotate_secret_channels` issues a new secret and invalidates the old one.
- `settings.verificationDelivery` = `contact`, unless the customer's application signs the customer in before the conversation opens; then they redact the six digits in their own logs.
