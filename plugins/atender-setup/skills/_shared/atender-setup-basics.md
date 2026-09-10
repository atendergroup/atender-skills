# Atender setup basics

Read this before any of the six setup skills. It carries what every area needs:
how to connect, who wins when sources disagree, the order of work, the preflight,
pacing, the report format, and the groundwork areas (Teams and tags, Knowledge
Base, Handbook) that no single skill owns.

Evidence date of the content in these skills: 2026-09-10. Anything marked
unconfirmed below is not a fact — check the live schema before you rely on it.

## What the Atender MCP is

Atender is a customer operations platform. The Atender MCP server exposes the
workspace configuration as tools: teams, tags, Knowledge Base, Handbook, Agent
Stacks, specialists, Capabilities, channels, phone numbers, queues and IVR flows.
Every call acts on the one workspace the connection belongs to.

- MCP server: `https://v3.atender.com/api/v1/mcp`
- Docs: `https://www.atender.com/docs`
- API reference: `https://www.atender.com/docs/api` and `https://v3.atender.com/api/openapi.json`
- Scopes: `https://v3.atender.com/.well-known/oauth-protected-resource`

## How to connect

Two ways, both against the same URL.

- Sign-in (OAuth): add the server as a connector or with `claude mcp add --transport http atender https://v3.atender.com/api/v1/mcp`, then authenticate in the browser.
- API key: a workspace key that starts with `sa_live_`, sent as `Authorization: Bearer sa_live_...`. Never ask the user to paste a key into the chat; tell them where in the app to enter it.

Scope map. Scope names do not follow one pattern.

| Work | Scope |
| --- | --- |
| Create a team | `teams:manage` (`teams:write` only edits) |
| Tags | `tags:*` |
| Knowledge Base and Handbook | `knowledge:*` |
| Agent Stacks, specialists, Capabilities, API connections, handover and verification settings | `agents:*` |
| Opening hours | `opening-hours:*` |
| Email channels, custom channels, phone numbers | `channels:*` (reading email domains needs `email:read`) |
| Chat widget | `widgets:*` |
| Call queues, IVR flows, voice settings | `voice:*` |
| SMS | `sms:*` |
| Reading test results back | `conversations:read` and `agents:read` |

There is no route that reads a connection's own scopes. Probe one read per area
and read the 403 text. Do not say a scope is missing until a call returns 403.

## Precedence

1. The live tool input schemas win for field names, shapes and values. Always.
2. These skills win over `describe_configuration_model` for the order of areas and for how to test.
3. Call `describe_configuration_model` first anyway, and tell the user where it disagrees with a skill.
4. Put today's date on every "this does not exist" finding. Probe again before you act on one more than a day old.

## Order of operations

Teams, then tags, then Knowledge Base and Handbook, then specialists, Agent Stack
and members, then personality, then Capabilities, then verification settings,
then opening hours, then handover, then channels, then the voice Agent Stack,
then call queues, IVR flow and phone number, then test conversations.

This order wins over any other order you read. Why each step is where it is:

- Teams first: a channel, a stack handover, a call queue and a phone number all take a team id.
- Tags before anything applies one: an unknown tag name is dropped with a 200.
- Knowledge Base and Handbook before the stack answers anybody.
- Capability: endpoint, then capability, then publish, then attach. A create with status published is refused.
- Verification settings before any Capability above the read tier.
- Opening hours before the opening-hours handover switch.
- Call queues before an IVR flow; the number after both. Without a queue the write answers 201 and the number never syncs.

If an area needs an earlier step that does not exist, add it to the plan or
report the item as blocked.

## Preflight: list what exists before you write anything

One read call per area, before the first write, in every area you are about to
touch: `list_teams`, `list_users`, `list_tags`, `list_kb_articles` (no status
filter, `includeArchived=true`, every page), `list_handbook`, `list_agent_stacks`,
`list_specialists`, `list_capabilities`, `list_api_definitions`, `list_channels`,
`list_email_channels`, `list_email_domains`, `list_chat_widgets`,
`list_voice_phone_numbers`, `list_voice_call_queues`, `list_ivr_flows`,
`list_opening_hours_rules`.

Then report three lists: reads that worked, write scopes the plan needs, and what
you could not check. A read does not prove write access. Do not filter a list you
use to decide what exists.

## The loop, for each area

- **Inspect.** List what exists. Match on the natural key (a name, a `slug`, an `externalId`), not on an id you did not create.
- **Ask.** Only for what the workspace cannot tell you.
- **Plan.** Show each create, change and delete with its fields and values. Wait for approval. Do not write before the user approves.
- **Apply.** Send only the fields you change. Never build a write body from a read: a write route refuses an unknown key with a 400 that names it, and a read returns computed fields. A list field replaces the list, so send the full list. For a full-replace write, read the object, merge your change, and send all of it. Do not send `null` unless you know it clears the field.
- **Verify.** Read back and compare field by field. A 200 is not proof; an updated timestamp can move on a no-op. Record every id you create.

**Never delete or overwrite an existing object unless the user says yes to that
object by name.** Never ask the user to paste a secret into the chat.

If a call fails, name the cause: our configuration, a missing scope, a missing
surface (a 404 can be a module that is off), or a platform fault. Do not invent
an operation. After 3 failed attempts, stop and tell the user.

## Pacing and cost

- At most one call every 2 seconds. Read `X-RateLimit-Remaining`. On a 429, wait for `Retry-After`, retry at most 3 times, then stop and tell the user. Specialist tests and documentation refreshes have a lower limit.
- A test call is a real call that costs money: get a yes before each one, wait 30 s after the last write, and do not score the first call.

## Report format

When you finish, and each time the user asks, report every checklist id, grouped
by area:

```
<Area>
- <ID>: done | skipped (reason) | blocked (reason)
```

For an inspect-only run (no writes at all), use `met | gap (what is missing) |
unreadable (why)` instead.

Then list the ids you created, the steps the user must do in the app, and the
open items with today's date. If something repeats or misfires, give the count.

---

## Groundwork: Teams and tags

Goal: every group that picks up work is a team, and every tag any area applies
exists before anything applies it.

Read first: `list_teams`, `list_users`, `list_tags`.

Needs from the customer: the groups of people who answer customers and who is in
each; the subjects they want to filter conversations by.

- **TT-01** Scopes: see the scope map above.
- **TT-02** One team per group that picks up work, named after the people, not a subject. At least one team, even empty.
- **TT-03** `create_teams` with `memberIds` from `list_users`, only for names not in `list_teams`.
- **TT-04** Change members only by the full-list merge in the loop above.
- **TT-05** Tags = the tags the areas in play apply, plus the customer's filter subjects. Short, in the team's language.
- **TT-06** `create_tags`: `name` (max 100, exact case), `description` (one line; the auto-tag model reads it). No update route exists, so the description is final. Create only names not in `list_tags`.

Rules:

- A second `create_tags` with an existing name answers 500. It made nothing. List again, do not retry.
- Tags made here never auto-tag. Tell the user which to switch on in the app.
- Teams cannot be deleted here. Tags can (`delete_tags`); ask first.
- `list_users` shows only users who are in a team. Say so if someone named is missing.
- KB tags are a different list. Never use them for conversations.

Verify: `list_teams` and `list_tags` — each planned name exists once, with its
members and description.

In the app: switch on AI auto-tagging per tag; add a user who is in no team;
delete a team.

## Groundwork: Knowledge Base

Goal: the articles customers read exist, published, in the knowledge base the
stack reads, taken from the customer's own sources, with the gaps listed.

Read first: `list_kb_articles` with no `status` filter, `includeArchived=true`,
every page.

Needs from the customer: where their customer-facing content lives today; the
questions they get most that have no article; whether customers need a hosted
help page.

- **KB-01** Partition: say which knowledge base each stack reads (`kbPartitionId`; null means the one with `isDefault: true`). Writes from here land only in the default one.
- **KB-02** Categories named after what a customer looks for. `import_kb_categories` (skips an existing slug); subcategories with `create_kb_subcategories`.
- **KB-03** One article per real question, whole, from the customer's source. Headings, paragraphs, lists, bold only. Subject in the heading; no "as described above"; site-only links as plain text.
- **KB-04** Key = `slug`. `import_kb_articles` with an explicit `slug`. A rerun skips it; without a slug it duplicates. Change content with `update_kb_articles` by id, and never change the title (it regenerates the slug).
- **KB-05** Only `published` articles reach the AI. Default is `draft`. Publish each approved article (`status: "published"` or `publish_kb_articles`).
- **KB-06** One article per question, in the primary language. Published articles are translated into each language active in the app (unconfirmed on 2026-09-10; check the live schema and one article before you rely on it). Never create one article per language.
- **KB-07** Hosted page: `update_kb_portal_settings` `kbTemplate` = `spotlight` or `sidebar`. Classic renders blank from here.
- **KB-08** List each topic with no source as a gap for the customer. Never write it yourself.

Verify:

- Table of every category and article: created, skipped or updated, and its status. Every approved article reads `published`.
- `search_knowledge_base` for three real customer questions returns the right article.
- Rerun the load: everything skips.
- Put each public article next to the Handbook rule on the same subject. Bring every conflict to the customer to rule on.
- Hosted page: open it. Blank with every call answering 200 means the template, not the content.

In the app: enable languages; restrict a specialist to some categories; fill a
second brand's knowledge base; edit the portal layout.

## Groundwork: Handbook

Knowledge Base is customer-facing and quotable. Handbook is internal instructions
the AI follows and never quotes.

Goal: the internal rules exist as visible entries, in the handbook the stack
reads, and every specialist that must follow them reads the Handbook.

Needs from the customer: what can be refunded, credited or waived without a
person, as numbers, and what happens above them; what is never said, and what is
said instead; the order of work for the situations they handle most.

- **HB-01** Handbook: the stack reads `handbookPartitionId`, or the one with `isDefault: true` when null. Categories take `handbookId`; entries follow their category.
- **HB-02** Categories: `import_handbook_categories` (skips an existing name).
- **HB-03** Entry: `keywords` (max 10), `visibility: true`, `externalId` = a stable key. Match on `externalId`.
- **HB-04** Change an entry with `update_handbook` by id. `import_handbook` skips an existing title and never updates.
- **HB-05** Write each entry as an instruction to the AI: a limit is a number and an action the specialist cannot take, with who owns the work instead; an escalation is a criterion; every "never say X" has its "say this instead". No customer wording: that is the Knowledge Base.
- **HB-06** Every specialist that must follow the Handbook reads `handbookEnabled: true` (the Agent Stacks text skill, SP-04, writes it).
- **HB-07** The source is a file the customer controls. Write the workspace from it. A live edit goes back into the file in the same session.
- **HB-08** Read back each entry where the customer's answer was vague and you chose the words.

Rules:

- `visibility: false` hides an entry from the AI. It is the only gate the AI reads. Never set it false to make an entry internal: every entry is internal.
- Access rules (confidential, scope to a stack) do not change what the AI reads today. Do not use them to restrict the AI.
- A rule in the Handbook and in specialist instructions is two sources. Report it and ask which one stays.

Verify:

- `list_handbook`: every entry once, `visibility: true`, `externalId` set, right `handbookId`.
- Every specialist that needs policy reads `handbookEnabled: true`.
- For every "do X" rule, a specialist on the stack holds a tool that does X. Show the gaps before you change wording.
- File and workspace match, field by field.

## Final verify, across the run

- Every checklist id reported, grouped by area.
- Every id you created, listed.
- Each planned object read back and compared field by field — not a success response.
- Every "do X" rule has a specialist that holds the tool for X.
- At least one proved conversation per stack, on a path that reaches no real customer.
- The steps the customer must do in the app, listed.
- The open items, dated.
