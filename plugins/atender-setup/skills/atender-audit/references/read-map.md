# What to read, area by area

One call per line, at most one every 2 seconds. Every line gets a result: a
value, a 403 (unreadable) or a 404 (a module that may be off).

## Groundwork

| Read | What it tells you |
| --- | --- |
| `list_teams` | Teams and their members |
| `list_users` | Only people already in a team |
| `list_tags` | Conversation tags, with their auto-tag fields |
| `list_opening_hours_rules` | Schedules, `timezone`, which is `isDefault` |
| `list_opening_hours_assignments` | Which team and channel pair uses which rule |
| `get_opening_hours` | The week as it stands |

## Knowledge Base and Handbook

| Read | What it tells you |
| --- | --- |
| `list_kb_partitions` | Which knowledge base is `isDefault` |
| `list_kb_categories`, `list_kb_subcategories` | The shape a customer navigates |
| `list_kb_articles` | Every article and its status. No `status` filter, `includeArchived=true`, every page |
| `list_kb_tags` | A separate list from conversation tags |
| `get_kb_portal_settings` | The portal look and `kbTemplate` |
| `list_handbook_partitions` | Which handbook is `isDefault` |
| `list_handbook_categories`, `list_handbook` | Entries, `visibility`, `externalId` |
| `list_handbook_access_rules_all` | Access rules, which do not change what the AI reads |

## Agent Stacks and specialists

| Read | What it tells you |
| --- | --- |
| `list_agent_stacks`, `get_agent_stacks` | `type`, `enabled`, `catchAllSpecialistId`, `kbPartitionId`, `handbookPartitionId`, `knowledgeBaseEnabled` (voice only), handover fields, personality, and the voice fields including `voiceLanguageVoices` |
| `list_agent_stacks_members` | Members and whether each is enabled |
| `get_agent_stack_orchestrator` | `systemPrompt` and `resolvedSource` |
| `list_agent_stack_prerequisites` | What the stack asks for before it answers |
| `list_specialists`, `get_specialists` | `enabled`, `kbEnabled`, `handbookEnabled`, `description`, `systemPrompt` |
| `list_specialists_tools` | Which tools each specialist holds |
| `list_specialists_playbooks`, `list_specialists_routing_rules` | Fixed sequences and routing |
| `list_verification_settings` | Which channels demand a verified customer |

## Capabilities and connections

| Read | What it tells you |
| --- | --- |
| `list_capabilities`, `get_capabilities` | Tier, status, who holds it |
| `list_api_definitions`, `get_api_definitions` | The customer's API, its auth, its documentation |
| `list_api_definitions_endpoints`, `list_api_definitions_endpoints_commands` | Endpoints and the commands built on them |
| `list_agent_tools`, `get_agent_tools` | The tools an assistant can call |
| `list_tool_execution_logs` | Whether a tool has run, and what it answered |

## Channels

| Read | What it tells you |
| --- | --- |
| `list_channels`, `get_channels` | Custom channels, `mainAgentId`, `isActive` |
| `list_email_channels` | Inboxes, `mainAgentId`, `teamId`, status |
| `list_email_domains` | Sending domains, and whether each record is `valid` |
| `list_chat_widgets`, `get_chat_widget` | `aliMainAgentId`, `defaultTeamId`, languages, messages |
| `list_channel_deliveries` | Whether a push channel is delivering |
| `get_sms_settings`, `list_sms_numbers` | `senderName` and the numbers |
| `list_custom_domains` | Hostnames and their status |

## Voice

| Read | What it tells you |
| --- | --- |
| `list_voice_settings` | Whether the voice feature is on |
| `list_voice_phone_numbers` | Numbers and what each is bound to |
| `list_voice_call_queues` | Queues, and whether there are any at all |
| `list_ivr_flows`, `get_ivr_flows` | The graph, its edges, `publishedRevision` |

## Brand and the rest

| Read | What it tells you |
| --- | --- |
| `list_branding` | Logo and colours |
| `get_email_brand_settings` | The brand on outgoing email |
| `list_incidents_settings`, `list_incidents_components` | The status page and its components |
| `get_csat_settings` | The survey, its switches and its look |
| `list_portal_settings` | The Customer Portal; 404 means the module is off |
| `get_default_signature` | The signature of the person whose key this is |
| `list_macros`, `list_snippets` | Canned wording that competes with the assistant's |
| `list_webhooks` | What the workspace pushes out |

## How to read a failure

| Answer | What it means in the report |
| --- | --- |
| 403 | Unreadable. Record the scope from the message |
| 404 on a settings route | The module is likely off, not a fault. Say which you think it is |
| Empty list | A real gap, not an unreadable |
| 429 | Wait for `Retry-After`, retry at most 3 times, then record it |
