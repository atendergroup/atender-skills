# The chat widget, and custom channels

## Widget fields

| Field | What it does |
| --- | --- |
| `aliMainAgentId` | The Agent Stack that answers. Without it the widget gets no AI answer |
| `defaultTeamId` | The team a conversation lands in, and the team whose opening hours the widget reads |
| languages | The languages the widget offers a visitor |
| `welcomeMessage` | The first thing a visitor reads, per language |
| `outsideHoursMessage` | What a visitor reads when the primary team is closed |
| colours | Widget branding. The rest of the brand is `update_branding` |

The widget is created with `create_chat_widget` and changed with
`update_chat_widget`. It cannot be deleted.

## Routing

A widget conversation goes to `defaultTeamId` and is answered by
`aliMainAgentId`. There is no per-language or per-page routing on the widget: to
send different pages to different teams, use more than one widget, or route
inside the Agent Stack with specialist routing rules.

## After hours

1. `create_opening_hours_rule` with `timezone` set explicitly — the default is UTC — and `holidayCountry`.
2. `set_opening_hours_assignment` for the primary team and the web chat channel.
3. `outsideHoursMessage` on the widget.

A team and channel pair with no assignment takes the default rule. No rule at all
reads as always open.

## The install snippet

```html
<script src="https://<app host>/chat-widget.js" data-widget-id="<id>" async></script>
```

No route returns this. Build it from the app host and the widget id from
`get_chat_widget`.

It answers as soon as it loads. Put it on a page nobody visits first — a staging
page, or a page behind a login — run the checks below, and only then hand it to
the customer for the real page.

## The test page

- Open the page and start a conversation. Record the widget session id at the moment you open it: an anonymous conversation is not on the conversation list, so without the id you cannot read the run back.
- Check the welcome message, the language, and that a reply arrives.
- Time the gap between the stored reply and the reply appearing on screen.
- Close the tab and reopen it: check what the visitor sees on a second visit.
- Check the after-hours message by testing outside the team's hours, or by moving the rule for the test.

## Custom channels

A custom channel carries a conversation inside the customer's own application.

1. `create_channels` with `mainAgentId` and `defaultTeamId`.
2. Give the customer the `secret` once — it is shown once. Never ask them to paste it back into the chat.
3. `test_channels` to prove the receiver before any message.
4. `create_channels_messages` to send an inbound message. The reply goes to the channel's `webhookUrl` only, so it reaches no customer.
5. `list_channel_deliveries` to read the result; `resend_channel_delivery` for a failed one.

Facts to hold on to:

- A push channel turns itself off after 10 failed deliveries in a row. A resend counts as a new delivery. `update_channels` `isActive: true` turns it on again.
- `rotate_secret_channels` issues a new secret and invalidates the old one.
- `settings.verificationDelivery` = `contact`, unless the customer's application signs the customer in before the conversation opens; then they redact the six digits in their own logs.
- A custom channel is the preferred path for test conversations, because no reply reaches a real customer.
