---
name: atender-workspace
description: Set up the ground an Atender workspace stands on over the Atender MCP server — a short company profile the other skills reuse, the brand applied to every surface that accepts it, and the groundwork of teams, tags, opening hours, the satisfaction survey, the SMS sender name and the email signature. Use this skill first, whenever a new customer starts; when the user says "set up my Atender workspace", "here is our brand guide", "apply our colours", "here is our logo", "create our teams", "our opening hours", "what should I tell the assistant about us"; and whenever an Atender MCP connection is present and nothing has been configured yet. What the assistant says belongs to atender-agent-stack-text.
---

# Atender workspace

## When to use

Use this as the first skill on a new workspace. It owns three things: a short
company profile the other skills read instead of asking the same questions
again, the brand on every surface a customer sees, and the groundwork that the
rest of the setup needs in place — teams, tags, opening hours, the satisfaction
survey, the SMS sender name and the email signature.

What the assistant says belongs to `atender-agent-stack-text`. Channels belong to
`atender-web-chat` and `atender-email`.

## Before you start

Read `../_shared/atender-setup-basics.md` first — connection, precedence, the
order of areas, the preflight, pacing, the report format and the Knowledge Base
and Handbook groundwork. Then call `describe_configuration_model`; branding and
the company profile are missing from it, so this skill wins on order.

Teams come before anything that takes a team id. The profile comes before the
questions the other skills would otherwise ask.

## Needs from the customer

- Whatever they already have written down: a brand guide, a zip of files, a folder, a website address, or answers in chat.
- The groups of people who answer customers, and who is in each.
- The subjects they want to filter conversations by.
- Opening hours per team and channel, with timezone and holiday country.
- Whether they want a satisfaction survey, and a name to send SMS from.

## Checklist: the company profile

- **WS-01** Read the material the customer hands over before asking anything. A website address counts: read the pages that say what the company sells and to whom.
- **WS-02** Write `atender-profile.md` in the working folder: company name, what it sells, who the customers are, languages and markets, tone in three words, words to avoid, what needs a person. One or two lines each.
- **WS-03** Show the profile and get a yes. Ask once for every line the material did not answer. Mark a line you guessed.
- **WS-04** Tell the other skills the file exists. They read it and do not ask those questions again.
- **WS-05** The personality takes `personality.context.businessDescription` and `personality.context.customerDescription` from this file, 2000 characters each, through `update_agent_stacks`. Use the customer's own words.

## Checklist: brand

- **WS-06** Pull the logo, the colours as hex and the fonts out of the material. Show the customer what was found, each value with where it came from, and get a yes before the first write.
- **WS-07** Look and feel: `update_branding` {`logoUrl`, `primaryColor`, `secondaryColor`, `darkPrimaryColor`, `darkSecondaryColor`, `linkColor`, `darkLinkColor`}.
- **WS-08** Email: `update_email_brand_settings` {`brandLogoUrl`, `brandPrimaryColor`, `brandSecondaryColor`, `defaultFromName`, `defaultFooterText`, `companyAddress`}. It merges, and it makes the row when there is none.
- **WS-09** Chat widget: `update_chat_widget` for `primaryColor`, `logoUrl` with `showLogo: true`, the bubble and launcher colours, and the dark set.
- **WS-10** Knowledge Base portal: `update_kb_portal_settings` {`logoUrl`, `faviconUrl`, `primaryColor`, `headerBackgroundColor`, `textColor`, `darkPrimaryColor`, `darkTextColor`} and `kbTemplate` = `spotlight` or `sidebar`.
- **WS-11** Status page: `update_incidents_settings` {`pageTitle`, `pageDescription`, `logoUrl`, `faviconUrl`, `primaryColor`, `showPoweredBy`}.
- **WS-12** Satisfaction survey: `update_csat_settings` {`logoUrl`, `primaryColor`, `backgroundColor`, `textColor`, `tenantName`}.
- **WS-13** Their own hostnames: `create_custom_domains` {`host`, `product`: `kb`, `status` or `portal`}. Give them the A record from `dnsRecords`, and call `create_custom_domains_verify` once after they publish it.

The field-by-field table for each surface, the colour patterns that differ, and
what no route accepts are in `references/brand-surfaces.md`.

## Checklist: groundwork

- **WS-14** Scopes: `teams:manage` to create a team, `tags:*`, `opening-hours:*`, `csat:write`, `sms:write`, `snippets:write`, `branding:write`, `domains:write`.
- **WS-15** One team per group that picks up work, named after the people, not a subject. At least one team, even empty.
- **WS-16** `create_teams` {`name`, `description`, `memberIds`}, only for names not in `list_teams`. A `memberId` who is not a member of the workspace gets a 400 that names the id.
- **WS-17** Change members only by the full-list merge in the basics file: `update_teams` replaces the list.
- **WS-18** `create_tags` {`name` (max 100, exact case), `description`}, only for names not in `list_tags`. The auto-tag model reads the description, and there is no update route, so the description is final.
- **WS-19** `create_opening_hours_rule` per schedule the customer keeps: `name`, `schedules`, `timezone` set explicitly, `holidayCountry`, `dateOverrides`. The first rule of a workspace becomes the default.
- **WS-20** `set_opening_hours_assignment` {`teamId`, `channel`, `ruleId`, `mode`} for each team and channel pair. A pair with no assignment takes the default rule, which is not the same as closed.
- **WS-21** Survey on or off: `update_csat_settings` {`enabled`, `sendSurveyEnabled`}. `enabled` cannot become true until `mailgunDomainId` names an active sending domain.
- **WS-22** SMS sender name: `update_sms_settings` {`senderName`}, 3 to 11 letters and digits with at least one letter. `null` puts every message back on the workspace number.
- **WS-23** Signature: `set_default_signature` writes the signature of the person whose API key this is, not the assistant's. Say so before you write it.
- **WS-24** Users: `list_users` only. Inviting a person happens in the app, and it must happen before that person can be a team member.

## Rules

- A colour is a hex string. A colour name or `rgb()` is a 400 on `update_branding`, `update_email_brand_settings` and `update_incidents_settings`.
- `null` clears a colour or a logo. Send it only when the customer asked for the value to go.
- Send only the fields you change. Never build a brand write body from a read.
- Fonts are not on this API at all.
- A second `create_tags` with an existing name answers 500 and makes nothing. List again; do not retry.
- Tags made here never auto-tag. `list_users` shows only people already in a team; say so when someone named is missing.
- Teams cannot be deleted here. Tags can, with `delete_tags`; ask first.
- Never overwrite a logo, colour, team or tag that already has a value unless the customer says yes to that object by name.

## Verify

Read every setting back with its own tool — `list_branding`,
`get_email_brand_settings`, `get_chat_widget`, `get_kb_portal_settings`,
`list_incidents_settings`, `get_csat_settings`, `list_custom_domains`,
`list_teams`, `list_tags`, `list_opening_hours_rules`,
`list_opening_hours_assignments`, `get_sms_settings`, `get_default_signature` —
and compare field by field with the plan.

Then show the customer one screen: each surface on a line, the logo and colours
it now carries, and the surfaces still on the product's own colours. Open the
Knowledge Base portal and the status page and look at them — a blank page while
every call answered 200 is the template, not the content. A person reads the SMS
sender name on a real handset; no route reports what a handset shows.

## What must be done in the app

Fonts. Inviting a user. Deleting a team. AI auto-tagging per tag. Uploading an
image file — those routes are multipart and are not MCP tools, so host the logo
yourself and pass the URL. The Knowledge Base portal layout, which
`layoutConfigV3` and `templateConfig` refuse here. Removing a custom domain,
which has no delete route. Publishing the DNS records at their own provider.
Tell the customer which of these are waiting on them.
