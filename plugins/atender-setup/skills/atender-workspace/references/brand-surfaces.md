# The brand, surface by surface

Each surface has its own tool and its own field names. Nothing is shared: a
colour set on one does not reach another.

## Look and feel

`list_branding` / `update_branding`.

| Field | What it is |
| --- | --- |
| `logoUrl` | Up to 2048 characters, or `null` |
| `primaryColor`, `secondaryColor`, `linkColor` | 6-digit hex, or `null` |
| `darkPrimaryColor`, `darkSecondaryColor`, `darkLinkColor` | The same three in dark mode |

`null` leaves the product's own colour. Anything but a 6-digit hex is a 400.

## Email

`get_email_brand_settings` / `update_email_brand_settings`. Merges the fields
you send; makes the row the first time.

| Field | What it is |
| --- | --- |
| `brandLogoUrl` | The logo at the top of an email |
| `brandPrimaryColor`, `brandSecondaryColor` | 6-digit hex, stored in lower case |
| `defaultFromName` | The name on an email |
| `defaultFromEmail` | The address; it does not make the address able to send |
| `defaultFooterText`, `companyAddress` | The footer and the postal address |
| `unsubscribeEnabled` | Whether an unsubscribe link is added |
| `defaultLanguage` | The language the wording is written in |

The sending domain is not here — that is `atender-email`.

## Chat widget

`get_chat_widget` / `update_chat_widget`, per widget. Set the dark values too,
or dark mode keeps the defaults.

`primaryColor`, `logoUrl` with `showLogo`, `receivedBubbleColor`,
`receivedBubbleTextColor`, `sentBubbleColor`, `sentBubbleTextColor`,
`chatBackgroundColor`, `sendButtonColor`, `launcherIconColor`,
`launcherBackgroundColor`, and the `dark…` version of each.

Some widget colour fields also accept a CSS name or `rgb()`. Send hex anyway.

## Knowledge Base portal

`get_kb_portal_settings` / `update_kb_portal_settings`. Name the knowledge base
in the `kbId` query parameter; a `kbId` in the body is a 400.

| Field | What it is |
| --- | --- |
| `logoUrl`, `faviconUrl`, `headerTitle` | The images and the header title |
| `primaryColor` | Hex, 3 to 8 digits, default `#3B82F6` |
| `headerBackgroundColor`, `textColor` | The header and the body text |
| `darkPrimaryColor`, `darkTextColor` | The dark set |
| `surfaceColor`, `pageBgColor` | The card and the page behind it, each with a `dark…` version |
| `kbTemplate` | `classic`, `sidebar` or `spotlight` |
| `seoTitle`, `metaDescription`, `ogImage` | What a search engine reads |

Use `spotlight` or `sidebar`. `classic` renders blank from here. A change to
`headerTitle`, `headerLinks` or `footerText` translates the portal chrome into
every active language. `layoutConfigV3` and `templateConfig` are refused: the
detailed layout is an app job.

## Status page

`list_incidents_settings` / `update_incidents_settings`.

| Field | What it is |
| --- | --- |
| `pageTitle`, `pageDescription` | Max 200 and max 2000 characters |
| `logoUrl`, `faviconUrl` | A full URL |
| `primaryColor` | 6-digit hex |
| `showPoweredBy` | Whether the Atender line stays at the foot |
| `subscriberFromChannelId` | One of the workspace's email channels, or `null` |

The host is not here: read it with `list_custom_domains`. `customDomains` in
this body is a 400.

## Satisfaction survey

`get_csat_settings` / `update_csat_settings`. Changes the fields you send.

| Field | What it is |
| --- | --- |
| `logoUrl` | The logo on the survey |
| `primaryColor` | 6-digit hex, default `#3B82F6` |
| `backgroundColor`, `textColor` | Default `#FFFFFF` and `#1F2937` |
| `tenantName` | The name the customer reads |
| `ratingDisplayType` | `numeric`, `emoji` or `stars` |
| `questionText` | The question |

## Their own hostnames

`create_custom_domains` {`host`, `product`}, where `product` is `kb`, `status`
or `portal`, and `kbId` names the knowledge base when the product is `kb`.

The domain starts at `pending_dns`. Give the customer the A record from
`dnsRecords`, then call `create_custom_domains_verify` once after they publish
it — rate-limited, up to 30 seconds, so call it on a change, not on a timer.

A hostname another workspace holds is a 409; one ending in an Atender domain is
a 400. Limits: 20 domains, 10 waiting on DNS. No delete route — removing a
domain is an app job.

<!-- site:skip -->
## Not on the API

| What | Where it happens |
| --- | --- |
| Fonts and typefaces | The app. No route carries a font field |
| Uploading an image file | Multipart routes exist on the REST API but are not MCP tools. Host the file and pass the URL |
| The Knowledge Base portal layout | The app; `layoutConfigV3` and `templateConfig` are refused |
| Removing a custom domain | The app |
| The chat widget launcher image file | The app |
| Deleting a team, inviting a user, AI auto-tagging per tag | The app |
