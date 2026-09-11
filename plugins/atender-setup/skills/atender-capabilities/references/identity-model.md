# The identity model

A Capability above the `read` tier must know which customer it is acting for, and
the answer may never come from the conversation. It comes from the verified
session.

Set all of this in `authModeConfig`, with `update_api_definitions`, before any
tier above `read` (CA-04).

<!-- site:skip -->
## `sessionVariableMapping`

Maps the verified session onto variables the request can use. A variable filled
this way is filled only after the customer has proved who they are — so the
specialist's Brief must say: verify first.

<!-- site:skip -->
## `claimVariableMapping`

After a one-time code, the claim holds the verified email address or the E.164
phone number the code was sent to. Map that claim onto whichever argument the
API uses to identify a person.

If the API cannot take an email or a phone number as the handle, the customer has
to give you an identifier the customer themselves holds, plus the endpoint that
lists the people on that account. Then check the live schema for
`accountVerification`; if it is present, set it. That field is unconfirmed as of
2026-09-10 — prove one bound call before you rely on it.

## `resourceOwnership`

- `resourceKeys` — the names of the **arguments** the model fills that name a resource. Not fields in the response.
- `lookup.path` — the endpoint that answers "does this person own this resource", with a `{{variable}}` for the identity.
- `lookup.identifierPaths` — where in that answer the owning identifier sits.

One connection holds one ownership rule. If two operations need different
ownership rules, they need two connections.

<!-- site:skip -->
## Redaction

`test_api_definitions_redaction` against a real sample response first, then write
`redactionRules`, `redactionApplyDefaults` and `redactionCustomFields`. Redaction
is what stops a response field the model never needed from reaching the customer.

<!-- site:skip -->
## Diagnosing an identity failure

1. Did the customer verify? On text channels `list_conversation_events` shows `verification_requested` and `verification_completed`. A call writes no verification event.
2. `list_tool_execution_logs` with `conversationId`, then `get_tool_execution_logs`: read the `inputPayload`. An empty identity argument means the mapping, not the model.
3. A refusal that names an attached, published Capability: check the assignment once with `list_specialists_tools`, then report a platform fault. Do not rewrite the Brief.
