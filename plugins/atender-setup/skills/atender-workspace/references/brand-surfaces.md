# The brand, surface by surface

Six surfaces carry the brand, and each has its own tool and its own field names.
Nothing is shared between them: a colour set on one does not reach another.

## Look and feel

`list_branding` reads it, `update_branding` writes it. This is the workspace's
own look, the one the Look & Feel screen shows.

| Field | What it is |
| --- | --- |
| `logoUrl` | Up to 2048 characters, or `null` |
| `primaryColor` | 6-digit hex, or `null` |
| `secondaryColor` | 6-digit hex, or `null` |
| `darkPrimaryColor` | The primary colour in dark mode |
| `darkSecondaryColor` | The secondary colour in dark mode |
| `linkColor` | The colour of a link |
| `darkLinkColor` | The colour of a link in dark mode |

A colour that is `null` leaves the product's own colour in place. Anything that
is not a 6-digit hex is a 400, because the value goes straight into the page.

## Email

`get_email_brand_settings` and `update_email_brand_settings`. It merges the
fields you send, and it makes the row the first time.

| Field | What it is |
| --- | --- |
| `brandLogoUrl` | The logo at the top of an email |
| `brandPrimaryColor` | 6-digit hex, stored in lower case |
| `brandSecondaryColor` | 6-digit hex, stored in lower case |
| `defaultFromName` | The name on an email |
| `defaultFromEmail` | The address; it does not make the address able to send |
| `defaultFooterText` | The footer |
| `companyAddress` | The postal address |
| `unsubscribeEnabled` | Whether an unsubscribe link is added |
| `defaultLanguage` | The language the wording is written in |

The domain a message leaves from is not here. That is `atender-email`.

## Chat widget

`get_chat_widget` and `update_chat_widget`, per widget. The widget carries a
full light and dark palette, so set both or the dark mode keeps the defaults.

`primaryColor`, `logoUrl` with `showLogo`, `receivedBubbleColor`,
`receivedBubbleTextColor`, `sentBubbleColor`, `sentBubbleTextColor`,
`chatBackgroundColor`, `sendButtonColor`, `launcherIconColor`,
`launcherBackgroundColor`, and the `dark…` version of each.

The widget's colour fields take more than hex — a CSS colour name or `rgb()`
passes the pattern on some of them. Send hex anyway, so the value reads the same
on every surface.

## Knowledge Base portal

`get_kb_portal_settings` and `update_kb_portal_settings`. Name the knowledge base
in the `kbId` query parameter; a `kbId` in the body is a 400.

| Field | What it is |
| --- | --- |
| `logoUrl`, `faviconUrl` | The images |
| `headerTitle` | The title in the header |
| `primaryColor` | Hex, 3 to 8 digits, default `#3B82F6` |
| `headerBackgroundColor`, `textColor` | The header and the body text |
| `darkPrimaryColor`, `darkTextColor` | The dark set |
| `surfaceColor`, `pageBgColor` | The card and the page behind it, with a `dark…` version of each |
| `kbTemplate` | `classic`, `sidebar` or `spotlight` |
| `seoTitle`, `metaDescription`, `ogImage` | What a search engine reads |

Use `spotlight` or `sidebar`. `classic` renders blank from here.

A change to `headerTitle`, `headerLinks` or `footerText` starts the translation
of the portal chrome into every active language, the same as a change made in
the app.

`layoutConfigV3` and `templateConfig` are refused. The detailed layout is an app
job.

## Status page

`list_incidents_settings` and `update_incidents_settings`.

| Field | What it is |
| --- | --- |
| `pageTitle` | Max 200 characters |
| `pageDescription` | Max 2000 characters |
| `logoUrl`, `faviconUrl` | A full URL |
| `primaryColor` | 6-digit hex |
| `showPoweredBy` | Whether the Atender line stays at the foot |
| `subscriberFromChannelId` | One of the workspace's email channels, or `null` |

The host the page is served from is not here. Read it with
`list_custom_domains`, and `customDomains` in this body is a 400.

## Satisfaction survey

`get_csat_settings` and `update_csat_settings`. It changes the fields you send;
an empty body changes nothing.

| Field | What it is |
| --- | --- |
| `logoUrl` | The logo on the survey |
| `primaryColor` | 6-digit hex, default `#3B82F6` |
| `backgroundColor` | Default `#FFFFFF` |
| `textColor` | Default `#1F2937` |
| `tenantName` | The name the customer reads |
| `ratingDisplayType` | `numeric`, `emoji` or `stars` |
| `questionText` | The question |

## Their own hostnames

`create_custom_domains` {`host`, `product`}, where `product` is `kb`, `status`
or `portal`, and `kbId` names the knowledge base when the product is `kb`.

The domain starts at `pending_dns`. Read `dnsRecords` from the answer, give the
customer the A record, and call `create_custom_domains_verify` once after they
publish it — the check is rate-limited and can take 30 seconds, so call it when
something changed, not on a timer.

A hostname another workspace holds is a 409. A hostname that ends in an Atender
domain is a 400. The limits are 20 domains, and 10 waiting on DNS.

There is no delete route. Removing a domain stays an app job.

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
