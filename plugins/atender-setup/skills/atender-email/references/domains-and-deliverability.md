# Email domains, DNS and deliverability

<!-- site:skip -->
## Provisioned domain or the customer's own

Atender provides a sending domain out of the box. Use it unless the customer
wants mail to come from their own domain. Their own domain costs a DNS change and
a verification round; the provisioned one costs nothing.

<!-- site:skip -->
## The MX warning

`create_email_domains` returns a record set that **includes MX**, and MX records
move **all** inbound mail for the name they are published on to Atender.

So: if the apex domain already receives mail — the company's own mailboxes, a
help desk, anything — do not register the apex. Register a subdomain instead,
such as `mail.example.com` or `help.example.com`, publish the returned records
on that name, and leave the apex untouched. The assistant then answers at
`support@mail.example.com`.

Say this to the customer in plain words before they publish anything. It is the
one change in this area that can take a company's mail down.

## The DNS records

`create_email_domains` {`domainName`, `fromEmail`, `fromName`} returns
`dnsRecords`. Give them to the customer as a table with four columns: type, host,
value, and what it is for.

| Type | What it is for | Care |
| --- | --- | --- |
| TXT (SPF) | Says Atender may send for the domain | **Merge into the existing SPF record.** A domain may have only one |
| CNAME or TXT (DKIM) | Signs outgoing mail | Copy the value exactly. Long values are often split by a DNS UI |
| MX | Receives mail for the domain | Only on a domain or subdomain that receives nothing else |
| TXT (DMARC) | Policy for SPF and DKIM failures | **Do not touch it.** If none exists, leave it to the customer |

Then, after they publish: `create_email_domains_verify`, once. Status must read
`active` and every record `valid`.

If a record still reads `invalid` after the TTL has passed, read the expected
value back from `get_email_domains` and compare character by character. A
trailing dot, a split TXT value, or an SPF record that was replaced rather than
merged are the usual causes.

<!-- site:skip -->
## The inbox, in the safe order

1. `create_email_channels` with `mainAgentId`, `teamId` and `status: "disabled"`.
2. Send one real email from an outside mailbox the customer controls.
3. Read the conversation and the AI reply in full.
4. Only then `update_email_channels` `status: "active"`.

An inbox created on a domain that is not `active` is refused with 400
`DOMAIN_INACTIVE`.

<!-- site:skip -->
## Reading deliverability back

- `list_email_deliverability_suppressions` — addresses Atender will not send to. A bounced or complained address lands here and stays until it is removed. This is the first thing to read when a reply never arrives.
- `list_email_deliverability_activity` — what happened to a given message: accepted, delivered, bounced, complained.
- `get_channel_delivery` and `list_channel_deliveries` — the delivery record for a push channel, not for email.

A reply that Atender stored is not a reply the customer received. Read the
activity, not the conversation.

## SMS sending

- `update_sms_settings` {`senderName`}: 3 to 11 letters and digits. Some countries ignore an alphanumeric sender and substitute a number; a person has to read it on a real handset.
- `update_sms_number_routing` takes `teamId` **or** `mainAgentId`, never both.
- `list_sms_numbers` shows what exists; `list_sms_messages` and `get_sms_message` read a message back, and `refresh_sms_message_status` re-reads its status from the carrier.
- Never set SMS `enabled` to false as a pause. It stops one-time codes too, so every customer verification on SMS fails silently.

<!-- site:skip -->
## Signatures

`set_default_signature` and `get_default_signature` are a **person's** signature
on their replies. They are not the AI's sign-off. The assistant's wording lives
in the Agent Stack personality, and there is no greeting or signature field
there — if the customer wants the assistant to sign off a particular way, it goes
in `personality.context.custom` as part of the reply shape.
