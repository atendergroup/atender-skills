---
name: atender-email
description: Set up or audit Atender email inboxes, sending domains and SMS sending over the Atender MCP server — use the provisioned domain or add the customer's own, publish and verify the DNS records, create the inbox disabled, test it, then make it active, and read deliverability back. Use this skill whenever the user asks to make Atender answer an email address; when they say "support@ our domain", "set up our inbox", "SPF and DKIM", "verify our email domain", "emails are not arriving", "replies go to spam", "MX records", "SMS sender name", or "the signature is wrong"; and whenever an Atender MCP connection is present and the work is about mail in or out. The chat widget and custom channels are the atender-web-chat skill.
---

# Atender email

## When to use

Use this when the customer wants an Atender assistant to answer an email address,
when mail is not arriving or not being delivered, or when they are naming
themselves as an SMS sender. It owns the sending domain, the DNS records, the
inbox, the safe order for switching it on, deliverability, and the SMS sender
name.

The chat widget and custom channels belong to `atender-web-chat`. What the
assistant says belongs to `atender-agent-stack-text`.

## Before you start

Read `../_shared/atender-setup-basics.md` first — connection, precedence, the
order of areas, the preflight, pacing, the report format and the groundwork
areas. Then call `describe_configuration_model` with `area: "web-chat-email"`;
this skill wins on order and on how to test.

The order is: teams, then the Agent Stack, then the domain, then verify, then the
inbox as `disabled`, then a test, then `active`. Never the other way round: an
active inbox on a live address answers a real customer with an untested stack.

Reading email domains needs `email:read`; the rest of this area needs
`channels:*`, and SMS needs `sms:*`.

## Needs from the customer

- The address the assistant should answer, and whether that domain already receives or sends mail somewhere else. If it receives mail, the sending domain is a subdomain such as `mail.example.com`.
- Who publishes their DNS.
- Which team and which Agent Stack the inbox belongs to.
- For SMS: a sender name, 3 to 11 letters and digits.

## Checklist

- **CH-01** For each channel, report its Agent Stack (`mainAgentId`), team and status. A channel with no stack gets no AI answer.
- **CH-02** Use the provisioned email domain unless the customer wants their own. For their own: `create_email_domains` {`domainName`, `fromEmail`, `fromName`}. **If the apex domain already receives mail, register a subdomain instead — `mail.example.com`, `help.example.com` — and leave the apex alone.** The records the call returns include MX, and MX records move all inbound mail for whatever name they are published on.
- **CH-03** Give the customer `dnsRecords` as a table. After they publish them, call `create_email_domains_verify` once: status `active`, every record `valid`. A domain reaches `active` on its sending records alone, so `active` does not prove that inbound mail arrives: the test email in Verify does.
- **CH-04** `create_email_channels` with `mainAgentId`, `teamId` and `status: "disabled"`. It is refused with 400 `DOMAIN_INACTIVE` until the domain is `active`. After the email test passes: `update_email_channels` `status: "active"`.
- **CH-08** SMS needs a number first. `request_sms_number` {`country`} asks Atender for one; it buys nothing, an Atender administrator reviews the request, buys the number and assigns it, and then `list_sms_numbers` shows it. `update_sms_settings` {`senderName`} is refused with 409 `SMS_NOT_SET_UP` while the workspace holds no SMS number. `update_sms_number_routing` takes `teamId` or `mainAgentId`, never both.
- **CH-09** Brand: `update_email_brand_settings`, `update_branding`, and `set_default_signature` where a person's signature is wanted.

The DNS record table, the delays and the deliverability reads are in
`references/domains-and-deliverability.md`.

## Rules

- Say the MX warning in CH-02 to the customer in plain words before they touch DNS. It is the one change here that can take a company's mail down.
- An inbox on a domain that is not `active` is refused with 400 `DOMAIN_INACTIVE`. Verify the domain first.
- Way out for a domain: `delete_email_domains`. A domain that is not verified and has no inbox is deleted, also at the email provider, and cannot be restored. A verified domain, or one with an inbox, is only disabled with all its inboxes. Read `outcome` (`deleted` or `disabled`) in the answer. Ask the customer first, by name.
- `set_default_signature` is a person's signature, not the AI's. The assistant's wording lives in the stack personality, and there is no greeting or signature field there.

## Delays

DNS propagation is not instant. Publish, wait, then call
`create_email_domains_verify` once — never poll it in a loop.

## Verify

- `get_email_domains`: status `active`, every record `valid`.
- `get_email_channels`: stack, team and status as planned. `disabled` until the test passes.
- Send one real email from an outside mailbox the customer controls. Show the conversation and the AI reply.
- If no reply arrives, read `list_email_deliverability_suppressions`, then `list_email_deliverability_activity` for the address.
- Only then `update_email_channels` `status: "active"`.
- SMS: read `get_sms_settings` back. A person reads the sender name on a real handset — no route reports what a handset shows.

## What must be done in the app

Publishing the DNS records at their own DNS provider. Entering any real
credential. Tell the customer which are waiting on them, and
give them the record table to hand to whoever runs their DNS.
