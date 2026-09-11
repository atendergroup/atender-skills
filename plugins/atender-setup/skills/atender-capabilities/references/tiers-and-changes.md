# Tiers, and changing a live Capability

## The tier table

`actionTier` on a Capability. Do not send `securityLevel`; it is derived.

| `actionTier` | What the operation does | Security level | What the customer must have proved |
| --- | --- | --- | --- |
| `read` | Reads nothing that belongs to one person — a price list, opening hours, a product catalogue | 0 | Nothing |
| `read_verified` | Reads one person's data — an order, an invoice, a booking | 2 | A one-time code |
| `act` | Changes one person's data — reschedules, updates an address, cancels | 2 | A one-time code |
| `transact` | Money — a refund, a charge, a credit | 3 | A signed-in customer. On a call it hands over instead |

Rules that follow from the table:

- Verification settings (CV-01 to CV-05 in the `atender-agent-stack-text` skill) come before any Capability above `read` goes on a stack.
- Give no level 2 Capability to a stack whose channel has no working method: voice with `voiceCallerVerificationEnabled` false, or text with `stepUpByCodeEnabled` false. The chat widget is the exception.
- `ownershipExempt: true` only where the operation names nothing a customer owns.
- Every specialist that holds a level 2 Capability carries the CV-07 wording in its `systemPrompt`.

## Changing a live Capability (CA-11)

| What changes | How | Live when |
| --- | --- | --- |
| Parameters or request body | `update_api_definitions_endpoints` | On the next publish of the Capability |
| `description` | `update_capabilities`, then `publish_capabilities` | After the publish |
| `usageGuidance` | `update_capabilities` | On the next turn |
| The connection (auth, ownership, redaction) | `update_api_definitions` | Unpublish and publish one Capability to re-run the checklist |
| Removing a tool from everyone | `unpublish_capabilities` | Immediately. The assignments stay |

After any of these, refresh each Playbook that quotes the Capability: send a
PATCH to the Playbook, then read `synthesizedProse` back. A Playbook belongs to
one specialist, and a copy drifts.

<!-- site:skip -->
## The publish checklist

`publish_capabilities` runs a checklist, but only on the transition to
`published`. A Capability that is already published does not re-run it when the
connection changes underneath it. Unpublish and publish one Capability to force
the check.

Fix each failed item and publish again. Do not work around a failure by lowering
the tier.
