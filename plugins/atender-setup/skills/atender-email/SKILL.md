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
areas. Then call `describe_configuration_model`; channels are missing from it
entirely today, so this skill wins on order and on how to test.

The order is: teams, then the Agent Stack, then the domain, then verify, then the
inbox as `disabled`, then a test, then `active`. Never the other way round: an
active inbox on a live address answers a real customer with an untested stack.

Reading email domains needs `email:read`; the rest of this area needs
`channels:*`, and SMS needs `sms:*`.

## Needs from the customer

- The address the assistant should answer, and whether that domain already receives or sends mail somewhere else.
- Who publishes their DNS.
- Which team and which Agent Stack the inbox belongs to.
- For SMS: a sender name, 3 to 11 letters and digits.

## Checklist

- **CH-01** For each channel, report its Agent Stack (`mainAgentId`), team and status. A channel with no stack gets no AI answer.
- **CH-02** Use the provisioned email domain unless the customer wants their own. For their own: `create_email_domains` {`domainName`, `fromEmail`, `fromName`}. **If that domain already has MX records, use a subdomain.**
- **CH-03** Give the customer `dnsRecords` as a table. After they publish them, call `create_email_domains_verify` once: status `active`, every record `valid`.
- **CH-04** `create_email_channels` with `mainAgentId`, `teamId` and `status: "disabled"`. After the email test passes: `update_email_channels` `status: "active"`.
- **CH-08** `update_sms_settings` {`senderName`}. `update_sms_number_routing` takes `teamId` or `mainAgentId`, never both.
- **CH-09** Brand: `update_email_brand_settings`, `update_branding`, and `set_default_signature` where a person's signature is wanted.

The DNS record table, the MX warning, the delays and the deliverability reads are
in `references/domains-and-deliverability.md`.

## Rules

- **Publishing MX records moves all inbound mail of that domain.** If the domain already receives mail anywhere, use a subdomain instead. Say this to the customer before they touch DNS.
- Merge the SPF value into the existing SPF record. A domain may have only one. Do not touch DMARC.
- An inbox on a domain that is not `active` is refused with 400 `DOMAIN_INACTIVE`. Verify the domain first.
- Never set SMS `enabled` to false as a pause. It stops one-time codes too.
- `set_default_signature` is a person's signature, not the AI's. The assistant's wording lives in the stack personality, and there is no greeting or signature field there.
- Publishing the DNS records happens in the customer's DNS, not here.
- Never delete or overwrite an inbox or a domain unless the customer says yes to that object by name.

## Delays

DNS propagation is not instant. Publish, wait, then call
`create_email_domains_verify` once — do not poll it in a loop. If a record still
reads `invalid` after the customer's TTL has passed, read the exact expected
value back from `get_email_domains` and compare it character by character with
what is published; a trailing dot or a split TXT value is the usual cause.

## Verify

- `get_email_domains`: status `active`, every record `valid`.
- `get_email_channels`: stack, team and status as planned. `disabled` until the test passes.
- Send one real email from an outside mailbox the customer controls. Show the conversation and the AI reply.
- If no reply arrives, read `list_email_deliverability_suppressions`, then `list_email_deliverability_activity` for the address.
- Only then `update_email_channels` `status: "active"`.
- SMS: read `get_sms_settings` back. A person reads the sender name on a real handset — no route reports what a handset shows.

## What must be done in the app

Publishing the DNS records the customer's own DNS provider holds. Requesting an
SMS number. Entering any real credential. Tell the customer which of these are
waiting on them, and give them the record table to hand to whoever runs their
DNS.
